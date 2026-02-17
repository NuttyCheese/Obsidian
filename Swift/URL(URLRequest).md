### 1. Что такое URL и URLRequest (коротко и чётко)

| Класс/Структура | Тип в Swift | Что это                                                | Основное назначение                                         | Когда создаётся           | Когда используется                             |
| --------------- | ----------- | ------------------------------------------------------ | ----------------------------------------------------------- | ------------------------- | ---------------------------------------------- |
| **URL**         | [[struct]]  | Универсальный адрес ресурса (Uniform Resource Locator) | Представляет **местоположение** (URL-строку)                | Из строки или компонентов | Как адрес для запроса, файла, deep link        |
| **URLRequest**  | [[class]]   | Полноценный [[HTTP]]-запрос                            | Содержит **URL** + метод + заголовки + тело + политику кэша | На основе URL             | Передаётся в URLSession для выполнения запроса |

**Короткое правило**:
> `URL` — это **адрес**.  
> `URLRequest` — это **готовый запрос** с этим адресом + всеми настройками (метод, заголовки, тело и т.д.).

### 2. Сравнение URL и URLRequest

| Характеристика                                                | URL                            | URLRequest                                                 |
| ------------------------------------------------------------- | ------------------------------ | ---------------------------------------------------------- |
| Тип                                                           | `struct`                       | `class`                                                    |
| Основная роль                                                 | Хранит адрес ресурса           | Хранит полный запрос (адрес + настройки)                   |
| Может быть nil                                                | Да (URL(string:) → опционал)   | Нет (инициализируется всегда)                              |
| Содержит метод ([[GET-HTTP\|GET]]/[[POST-HTTP\|POST]] и т.д.) | Нет                            | Да (`httpMethod`)                                          |
| Содержит заголовки                                            | Нет                            | Да (`allHTTPHeaderFields`, `setValue:forHTTPHeaderField:`) |
| Содержит тело запроса                                         | Нет                            | Да (`httpBody`, `httpBodyStream`)                          |
| Содержит политику кэша                                        | Нет                            | Да (`cachePolicy`)                                         |
| Содержит таймаут                                              | Нет                            | Да (`timeoutInterval`)                                     |
| Используется в [[URLSession]]                                 | Косвенно (через URLRequest)    | Прямо (`dataTask(with:)`, `data(for:)`)                    |
| Можно мутировать                                              | Нет (immutable)                | Да (var request = URLRequest(...))                         |
| Поддерживает deep linking                                     | Да (scheme, host, path, query) | Да (но чаще используется для HTTP)                         |

### 3. Создание и использование URL

#### 3.1 Простейшее создание

```swift
if let url = URL(string: "https://api.example.com/v1/users") {
    print(url.absoluteString) // https://api.example.com/v1/users
}
```

#### 3.2 Безопасное построение через [[URLComponent]]

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "api.example.com"
components.path = "/v1/users"
components.queryItems = [
    URLQueryItem(name: "sort", value: "name"),
    URLQueryItem(name: "limit", value: "50")
]

if let url = components.url {
    print(url) // https://api.example.com/v1/users?sort=name&limit=50
}
```

#### 3.3 Локальные файлы

```swift
let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
let fileURL = documents.appending(path: "cache/data.json")
print(fileURL.path) // /Users/.../Documents/cache/data.json
```

#### 3.4 Добавление пути и параметров

```swift
let base = URL(string: "https://api.example.com")!
let userURL = base.appending(path: "v1/users/123")
let searchURL = userURL.appending(queryItems: [URLQueryItem(name: "expand", value: "profile")])
```

### 4. Создание и настройка URLRequest

#### 4.1 Базовый GET-запрос

```swift
guard let url = URL(string: "https://api.example.com/users") else { return }

var request = URLRequest(url: url)
request.httpMethod = "GET"

let task = URLSession.shared.dataTask(with: request) { data, response, error in
    // обработка
}
task.resume()
```

#### 4.2 Современный способ ([[async]]/[[await]] + [[Codable]])

```swift
func fetchUsers() async throws -> [User] {
    guard let url = URL(string: "https://api.example.com/users") else {
        throw URLError(.badURL)
    }
    
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return try JSONDecoder().decode([User].self, from: data)
}
```

#### 4.3 POST с [[JSON]]

```swift
struct CreateUser: Encodable {
    let name: String
    let email: String
}

func createUser(name: String, email: String) async throws {
    let payload = CreateUser(name: name, email: email)
    
    guard let url = URL(string: "https://api.example.com/users") else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

#### 4.4 Добавление заголовков и аутентификации

```swift
var request = URLRequest(url: url)
request.httpMethod = "GET"
request.setValue("application/json", forHTTPHeaderField: "Accept")
request.setValue("Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...", forHTTPHeaderField: "Authorization")
request.timeoutInterval = 30
request.cachePolicy = .reloadIgnoringLocalCacheData
```

### 5. Сравнение: когда использовать URL vs URLRequest

| Ситуация                                                   | Используй URL | Используй URLRequest | Почему                                        |
| ---------------------------------------------------------- | ------------- | -------------------- | --------------------------------------------- |
| Нужно просто открыть ссылку                                | Да            | Нет                  | `UIApplication.shared.open(url)`              |
| Нужен [[deep link]] / [[Universal Link]]                   | Да            | Нет                  | `onOpenURL`, `userActivity.webpageURL`        |
| Нужно выполнить GET-запрос без настроек                    | Да            | Косвенно             | `URLSession.shared.data(from: url)`           |
| Нужен POST/[[PUT-HTTP\|PUT]]/[[PATCH-HTTP\|PATCH]] с телом | Нет           | Да                   | Только URLRequest имеет httpBody              |
| Нужно добавить заголовки (Auth, Content-Type)              | Нет           | Да                   | URLRequest имеет setValue:forHTTPHeaderField: |
| Нужен кастомный таймаут / cache policy                     | Нет           | Да                   | URLRequest имеет timeoutInterval, cachePolicy |
| Нужен [[multipart or form-data]] (файлы)                   | Нет           | Да                   | URLRequest + httpBody с boundary              |

### 6. Типичные ошибки и как их избежать

| Ошибка                            | Последствия                           | Как избежать               |
| --------------------------------- | ------------------------------------- | -------------------------- |
| Конкатенация строк вручную        | Потеря /, ?, &, некорректный encoding | Только URLComponents       |
| [[Force unwrap]] `URL(string:)`   | Crash при некорректном URL            | [[if let]] / [[guard let]] |
| Нет проверки `scheme == "https"`  | Уязвимость, блокировка ATS            | Проверять scheme           |
| Забыли установить Content-Type    | Сервер не понимает тело               | Всегда явно указывать      |
| Использование `try!` в production | Crash при ошибке                      | `do-try-catch`             |
| Неправильный разбор query         | Потеря параметров                     | `URLComponents.queryItems` |

### 7. Лучшие практики 2026 года

- Никогда не строй URL через `+` — только `URLComponents`
- Всегда проверяй `if let url = URL(string: …)` или `guard let`
- Для API — **HTTPS** + **Codable** + **async/await**
- Для deep linking — **Universal Links** по HTTPS
- Для файлов — `FileManager.urls(for:in:)` + `appending(path:)`
- Для безопасности — проверяй `scheme == "https"` и `host`
- Для отладки — логируй `url.absoluteString` и `url.queryItems`
- Для больших запросов — используй `URLSessionConfiguration` с таймаутами и кэшем

**Короткий девиз**:
> «URL — это адрес. URLRequest — это письмо с адресом, маркой, заголовками и содержимым.  
> Адрес строишь через URLComponents → письмо отправляешь через URLRequest → всё безопасно и без багов.»
