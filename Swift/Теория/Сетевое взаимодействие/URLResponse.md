**`URLResponse`** — базовый класс в [[Foundation]], представляющий **ответ от сетевого запроса** (или локального ресурса).  
Он содержит базовую информацию: [[URL]], MIME-тип, ожидаемую длину данных, кодировку текста.

**`HTTPURLResponse`** — это **подкласс** `URLResponse`, который используется **только для HTTP/HTTPS-запросов** и добавляет:

- статус-код (statusCode)  
- все [[HTTP]]-заголовки (allHeaderFields)  
- статус-текст (status code как строка, редко используется)

### 1. Иерархия и когда какой класс используется

| Класс                 | Наследуется от | Когда используется                      | Содержит статус-код? | Содержит заголовки? | Пример использования  |
| --------------------- | -------------- | --------------------------------------- | -------------------- | ------------------- | --------------------- |
| `URLResponse`         | [[NSObject]]   | Общий ответ (HTTP, FTP, file, [[Data]]) | Нет                  | Нет                 | Редко напрямую        |
| **`HTTPURLResponse`** | `URLResponse`  | **Любой [[HTTP]]/[[HTTPS]]-запрос**     | Да                   | Да                  | 99% случаев в [[iOS]] |

**Правило 2026 года**:  
В 99,9% случаев ты работаешь именно с `HTTPURLResponse`.  
Всегда приводи `response as? HTTPURLResponse`.

### 2. Ключевые свойства URLResponse

| Свойство                | Тип         | Что возвращает                                    | Пример значения                 | Когда полезно      |
| ----------------------- | ----------- | ------------------------------------------------- | ------------------------------- | ------------------ |
| `url`                   | `URL?`      | Исходный URL запроса                              | `https://api.example.com/users` | Проверка редиректа |
| `mimeType`              | [[String]]? | MIME-тип ответа (application/json и т.д.)         | "application/json"              | Проверка формата   |
| `expectedContentLength` | `Int64`     | Ожидаемая длина данных в байтах (-1 = неизвестно) | 1024, -1                        | Прогресс-бар       |
| `textEncodingName`      | `String?`   | Кодировка текста (utf-8, utf-16 и т.д.)           | "utf-8"                         | Парсинг текста     |

### 3. Ключевые свойства HTTPURLResponse (самое важное)

| Свойство            | Тип                  | Что возвращает                               | Пример значения                      | Когда критично                     |
| ------------------- | -------------------- | -------------------------------------------- | ------------------------------------ | ---------------------------------- |
| `statusCode`        | [[Int]]              | HTTP-статус (200, 404, 500 и т.д.)           | 200, 401, 429                        | Проверка успеха                    |
| `allHeaderFields`   | `[AnyHashable: Any]` | Все HTTP-заголовки                           | ["Content-Type": "application/json"] | Cache-Control, ETag, Authorization |
| `url`               | `URL?`               | Финальный URL после редиректов               | Может отличаться от исходного        | Отслеживание редиректов            |
| `suggestedFilename` | `String?`            | Предложенное имя файла (Content-Disposition) | "report.pdf"                         | Скачивание файлов                  |

### 4. Полный пример обработки ответа ([[async]]/[[await]] + [[Codable]])

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

func fetchUser(id: Int) async throws -> User {
    guard let url = URL(string: "https://api.example.com/users/\(id)") else {
        throw URLError(.badURL)
    }
    
    let (data, response) = try await URLSession.shared.data(from: url)
    
    // Самое важное — проверка HTTPURLResponse
    guard let httpResponse = response as? HTTPURLResponse else {
        throw URLError(.badServerResponse)
    }
    
    // Проверка статуса
    switch httpResponse.statusCode {
    case 200...299:
        break // успех
    case 401:
        throw URLError(.userAuthenticationRequired)
    case 404:
        throw URLError(.fileDoesNotExist)
    case 429:
        throw URLError(.timedOut) // rate limit
    default:
        throw URLError(.badServerResponse)
    }
    
    // Проверка MIME-типа (опционально)
    if let mime = httpResponse.mimeType, !mime.contains("json") {
        throw URLError(.cannotParseResponse)
    }
    
    // Декодирование
    return try JSONDecoder().decode(User.self, from: data)
}
```

### 5. Реальные сценарии в [[iOS]]-разработке (2026)

#### Сценарий 1 — Проверка редиректов

```swift
let (data, response) = try await URLSession.shared.data(from: url)

if let httpResponse = response as? HTTPURLResponse {
    if httpResponse.url != url {
        print("Произошёл редирект с \(url) → \(httpResponse.url?.absoluteString ?? "")")
    }
    
    if httpResponse.statusCode == 301 || httpResponse.statusCode == 302 {
        print("Постоянный/временный редирект")
    }
}
```

#### Сценарий 2 — Работа с заголовками (ETag, Cache-Control)

```swift
if let httpResponse = response as? HTTPURLResponse {
    let headers = httpResponse.allHeaderFields
    
    if let etag = headers["ETag"] as? String {
        print("ETag для кэширования:", etag)
        // Сохранить ETag для следующего запроса
    }
    
    if let cacheControl = headers["Cache-Control"] as? String {
        print("Cache-Control:", cacheControl)
        // "max-age=3600, public"
    }
}
```

#### Сценарий 3 — Скачивание файла с проверкой Content-Disposition

```swift
func downloadFile(from url: URL) async throws {
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    // Получение имени файла из заголовков
    let suggestedName = httpResponse.suggestedFilename ?? "file"
    print("Предложенное имя файла:", suggestedName)
    
    // Сохранение
    let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
    let fileURL = documents.appendingPathComponent(suggestedName)
    try data.write(to: fileURL)
}
```

#### Сценарий 4 — Обработка ошибок по статус-кодам

```swift
func handleResponse(_ response: URLResponse?, data: Data?, error: Error?) throws -> Data {
    if let error = error {
        throw error
    }
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw URLError(.badServerResponse)
    }
    
    switch httpResponse.statusCode {
    case 200...299:
        guard let data = data else { throw URLError(.badServerResponse) }
        return data
        
    case 401:
        throw AuthError.unauthorized
        
    case 403:
        throw AuthError.forbidden
        
    case 404:
        throw URLError(.fileDoesNotExist)
        
    case 429:
        throw RateLimitError()
        
    default:
        throw URLError(.badServerResponse)
    }
}
```

### 6. Таблица: самые полезные свойства и методы

| Свойство / Метод                        | Тип                              | Что возвращает                              | Когда критично |
|-----------------------------------------|----------------------------------|---------------------------------------------|----------------|
| `statusCode` (только HTTPURLResponse)   | `Int`                            | 200, 404, 500 и т.д.                        | Проверка успеха |
| `allHeaderFields`                       | `[AnyHashable: Any]`             | Все заголовки ответа                        | Cache-Control, ETag, Content-Type |
| `mimeType`                              | `String?`                        | MIME-тип (application/json и т.д.)          | Проверка формата |
| `expectedContentLength`                 | `Int64`                          | Ожидаемая длина тела                        | Прогресс-бар |
| `suggestedFilename`                     | `String?`                        | Имя файла из Content-Disposition            | Скачивание файлов |
| `url`                                   | `URL?`                           | Финальный URL после редиректов              | Отслеживание редиректов |

### 7. Типичные ошибки и как их избежать

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Не приводить response к HTTPURLResponse     | Нет доступа к statusCode и заголовкам        | Всегда `as? HTTPURLResponse` |
| Игнорирование statusCode                    | Принимаются ошибочные ответы                 | Проверять 200...299 |
| Нет обработки редиректов                    | Неправильный URL в данных                    | Сравнивать `response.url` с исходным |
| Забыть проверить mimeType                   | Парсинг не-JSON как JSON → краш              | `if response.mimeType?.contains("json") == true` |
| Использование force unwrap                  | Crash при nil                                | `guard let httpResponse = response as? HTTPURLResponse` |

### 8. Лучшие практики 2026 года

- Всегда приводи `response as? HTTPURLResponse`
- Проверяй `statusCode` в диапазоне 200...299
- Обрабатывай типичные коды: 401, 403, 404, 429
- Проверяй `mimeType` перед декодированием
- Используй `suggestedFilename` при скачивании файлов
- Для отладки — логируй `response.url`, `statusCode`, `allHeaderFields["Cache-Control"]`, `allHeaderFields["ETag"]`
- Для продакшена — создавай кастомные ошибки (`enum NetworkError: Error`)
- Для прогресса — используй `expectedContentLength`

**Короткий девиз**:
> «URLResponse — это конверт с ответом.  
> HTTPURLResponse — конверт + письмо с номером ошибки и инструкциями.  
> Всегда открывай конверт как HTTPURLResponse и читай statusCode первым делом.»
