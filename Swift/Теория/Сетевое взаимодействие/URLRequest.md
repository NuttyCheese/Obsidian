**`URLRequest`** — это **класс** в [[Foundation]], представляющий **полноценный [[HTTP]]/[[HTTPS]]-запрос**, который можно отправить через [[URLSession]].

В отличие от `URL` (который хранит только адрес), `URLRequest` содержит **всё необходимое для выполнения запроса**:

- адрес ([[URL]])  
- метод ([[GET-HTTP|GET]], [[POST-HTTP|POST]], [[PUT-HTTP|PUT]], [[PATCH-HTTP|PATCH]], [[DELETE-HTTP|DELETE]])  
- заголовки (`Authorization`, `Content-Type`, `Accept` и т.д.)  
- тело запроса (`httpBody`)  
- политику кэширования (`cachePolicy`)  
- таймаут (`timeoutInterval`)  
- и другие параметры

### 1. Основные свойства и методы URLRequest

| Свойство / Метод                  | Тип                      | Что делает / возвращает                             | Изменяемое? | Пример использования                                            |
| --------------------------------- | ------------------------ | --------------------------------------------------- | ----------- | --------------------------------------------------------------- |
| `url`                             | `URL?`                   | Адрес запроса                                       | Да          | `request.url = url`                                             |
| `httpMethod`                      | [[String]]?              | Метод запроса (GET, POST, PUT, PATCH, DELETE)       | Да          | `request.httpMethod = "POST"`                                   |
| `httpBody`                        | [[Data]]?                | Тело запроса ([[JSON]], form-data, бинарные данные) | Да          | `request.httpBody = jsonData`                                   |
| `allHTTPHeaderFields`             | `[String: String]?`      | Все заголовки запроса                               | Да          | `request.allHTTPHeaderFields`                                   |
| `setValue(_:forHTTPHeaderField:)` | `Void`                   | Устанавливает или перезаписывает заголовок          | —           | `setValue("Bearer token", forHTTPHeaderField: "Authorization")` |
| `cachePolicy`                     | `URLRequest.CachePolicy` | Политика кэширования                                | Да          | `.returnCacheDataElseLoad`                                      |
| `timeoutInterval`                 | [[TimeInterval]]         | Максимальное время ожидания ответа                  | Да          | `30.0` (секунд)                                                 |
| `httpShouldHandleCookies`         | [[Bool]]                 | Разрешить автоматическую работу с cookies           | Да          | Обычно `true`                                                   |
| `httpShouldUsePipelining`         | `Bool`                   | Включить HTTP pipelining (редко)                    | Да          | По умолчанию `false`                                            |
| `mainDocumentURL`                 | `URL?`                   | URL основного документа (для cookies, referrer)     | Да          | Редко                                                           |

### 2. Сравнение: URL vs URLRequest vs [[URLSessionConfiguration]]

| Характеристика                  | URL                                      | URLRequest                                      | URLSessionConfiguration                          |
|---------------------------------|------------------------------------------|-------------------------------------------------|--------------------------------------------------|
| Что хранит                      | Только адрес                             | Полный запрос (URL + метод + заголовки + тело) | Настройки сессии (кэш, таймаут, cookies, proxy) |
| Можно ли изменить               | Нет (immutable)                          | Да (var request = ...)                          | Да (var config = ...)                            |
| Содержит httpMethod             | Нет                                      | Да                                              | Нет (но влияет на все запросы сессии)            |
| Содержит httpBody               | Нет                                      | Да                                              | Нет                                              |
| Содержит заголовки              | Нет                                      | Да                                              | Да (default headers для всех запросов)           |
| Содержит cachePolicy            | Нет                                      | Да (на уровне запроса)                          | Да (на уровне сессии)                            |
| Используется в URLSession       | Косвенно                                 | Прямо (`dataTask(with: request)`)               | При создании `URLSession(configuration:)`        |
| Пример использования            | `URL(string:)`                           | `URLRequest(url:)` + настройка                  | `URLSessionConfiguration.default`                |

### 3. Создание URLRequest — все варианты

#### 3.1 Базовый GET-запрос (самый частый)

```swift
guard let url = URL(string: "https://api.example.com/users") else { return }

var request = URLRequest(url: url)
request.httpMethod = "GET"  // по умолчанию уже GET, можно не писать

// Отправка
let task = URLSession.shared.dataTask(with: request) { data, response, error in
    // обработка
}
task.resume()
```

#### 3.2 Современный способ: [[async]]/[[await]] + [[Codable]] (2026 стандарт)

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
    
    var request = URLRequest(url: url)
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return try JSONDecoder().decode(User.self, from: data)
}
```

#### 3.3 POST-запрос с [[JSON]]

```swift
struct CreateUser: Encodable {
    let name: String
    let email: String
    let password: String
}

func createUser(_ user: CreateUser) async throws -> User {
    guard let url = URL(string: "https://api.example.com/users") else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(user)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

#### 3.4 PATCH-запрос (частичное обновление)

```swift
struct UpdateUser: Encodable {
    let name: String?
    let bio: String?
}

func updateUser(id: Int, name: String? = nil, bio: String? = nil) async throws {
    let payload = UpdateUser(name: name, bio: bio)
    
    guard let url = URL(string: "https://api.example.com/users/\(id)") else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.httpMethod = "PATCH"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
}
```

#### 3.5 Аутентификация (Bearer Token)

```swift
func fetchProtectedData(token: String) async throws -> Data {
    guard let url = URL(string: "https://api.example.com/protected") else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return data
}
```

#### 3.6 Отправка [[multipart or form-data]] (файлы + поля)

```swift
func uploadImage(_ imageData: Data, userId: Int) async throws {
    let boundary = "Boundary-\(UUID().uuidString)"
    let url = URL(string: "https://api.example.com/upload")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    
    // Поле userId
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"userId\"\r\n\r\n".data(using: .utf8)!)
    body.append("\(userId)\r\n".data(using: .utf8)!)
    
    // Файл
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"file\"; filename=\"photo.jpg\"\r\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
    body.append(imageData)
    body.append("\r\n".data(using: .utf8)!)
    
    body.append("--\(boundary)--\r\n".data(using: .utf8)!)
    
    request.httpBody = body
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

### 4. Таблица: ключевые свойства и методы URLRequest

| Свойство / Метод                        | Тип                              | Что делает / возвращает                              | Изменяемое? | Пример использования |
|-----------------------------------------|----------------------------------|------------------------------------------------------|-------------|----------------------|
| `url`                                   | `URL?`                           | Адрес запроса                                        | Да          | `request.url = url`  |
| `httpMethod`                            | `String?`                        | Метод запроса (GET, POST, PUT, PATCH, DELETE)        | Да          | `request.httpMethod = "POST"` |
| `httpBody`                              | `Data?`                          | Тело запроса                                         | Да          | `request.httpBody = jsonData` |
| `allHTTPHeaderFields`                   | `[String: String]?`              | Все заголовки                                        | Да          | `request.allHTTPHeaderFields` |
| `setValue(_:forHTTPHeaderField:)`       | `Void`                           | Устанавливает заголовок                              | —           | `setValue("Bearer token", forHTTPHeaderField: "Authorization")` |
| `cachePolicy`                           | `URLRequest.CachePolicy`         | Политика кэширования                                 | Да          | `.reloadIgnoringLocalCacheData` |
| `timeoutInterval`                       | `TimeInterval`                   | Таймаут запроса                                      | Да          | `30.0` (секунд)      |
| `httpShouldHandleCookies`               | `Bool`                           | Автоматическая работа с cookies                      | Да          | Обычно `true`        |

### 5. Типичные ошибки и как их избежать

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Нет установки `httpMethod`                  | По умолчанию GET → ошибка при POST/PUT       | Всегда явно указывать `request.httpMethod` |
| Нет `Content-Type` при отправке тела        | Сервер не понимает формат                    | Всегда `setValue("application/json", forHTTPHeaderField: "Content-Type")` |
| Force unwrap `URL(string:)`                 | Crash при некорректном URL                   | `guard let url = URL(string: …)` |
| Забыли `https`                              | ATS-блокировка или утечка данных             | Всегда проверять scheme |
| Использование `try!` в production           | Crash при ошибке сериализации                | `do-try-catch` |
| Отправка большого тела без `httpBodyStream` | Память переполняется                         | Для больших файлов — `httpBodyStream` |

### 6. Лучшие практики 2026 года

- Всегда используй **HTTPS**
- Модели — **Codable** ([[Encodable]] для запроса, [[Decodable]] для ответа)
- Предпочитай **async/await** + `URLSession.shared.data(for:)`
- Устанавливай заголовки: `Accept: application/json`, `Content-Type: application/json`
- Обрабатывай статус-коды (200–299 — успех, 400–499 — клиентская ошибка, 500+ — серверная)
- Для аутентификации — Bearer Token в `Authorization`
- Для файлов — `multipart/form-data` с `httpBody`
- Для отладки — логируй `request.url?.absoluteString`, `request.allHTTPHeaderFields`, `request.httpBody`
- Для больших запросов — кастомный `URLSessionConfiguration` с таймаутами и кэшем

**Короткий девиз**:
> «URLRequest — это письмо: адрес (URL) + марка (метод) + заголовки + содержимое (httpBody).  
> Безопасно, типобезопасно и современно — через async/await + Codable + HTTPS.»
