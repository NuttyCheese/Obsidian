**URL** (Uniform Resource Locator) — это **структура** ([[struct]]) в [[Swift]]/[[Foundation]], представляющая **универсальный адрес ресурса** в сети или локальной файловой системе.

Это **единственный** рекомендованный способ работы с адресами в Swift (никаких конкатенаций строк вручную!).

### 1. Зачем нужен URL вместо строки

| Проблема при работе со строкой | Решение через URL |
|--------------------------------|--------------------|
| Ошибки при конкатенации (потеря /, ? и т.д.) | URLComponents строит безопасно |
| Некорректные символы в query | Автоматическое percent-encoding |
| Сложный разбор пути/query/fragment | Готовые свойства: .host, .path, .query, .fragment |
| Нет проверки валидности | `URL(string:)` → nil при ошибке |
| Небезопасно для deep link | URL + scheme/host/path проверяются |

### 2. Создание URL — все варианты

#### Вариант 1 — из строки (самый частый)

```swift
if let url = URL(string: "https://api.example.com/v1/users/123?sort=desc&page=2#profile") {
    // безопасно
} else {
    print("Некорректный URL")
}
```

#### Вариант 2 — из компонентов (рекомендуется для динамических URL)

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "api.example.com"
components.path = "/v1/users"
components.port = 443
components.queryItems = [
    URLQueryItem(name: "sort", value: "desc"),
    URLQueryItem(name: "page", value: "2"),
    URLQueryItem(name: "limit", value: "50")
]

if let url = components.url {
    print(url.absoluteString)
    // https://api.example.com/v1/users?sort=desc&page=2&limit=50
}
```

#### Вариант 3 — локальный файл

```swift
let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
let fileURL = documents.appendingPathComponent("report.pdf")
// или
let fileURL = URL(fileURLWithPath: "/Users/user/Downloads/image.jpg")
```

#### Вариант 4 — relative URL (относительный)

```swift
let base = URL(string: "https://example.com/api/v1/")!
let relative = URL(string: "users/123", relativeTo: base)!
print(relative.absoluteString) // https://example.com/api/v1/users/123
```

### 3. Все свойства и методы URL

| Свойство / Метод                   | Тип                    | Что возвращает                            | Пример использования      |
| ---------------------------------- | ---------------------- | ----------------------------------------- | ------------------------- |
| `absoluteString`                   | [[String]]             | Полный URL как строка                     | логирование, отладка      |
| `scheme`                           | String?                | Протокол (https, file, myapp)             | проверка [[deep link]]    |
| `host`                             | String?                | Домен (api.example.com)                   | проверка домена           |
| `port`                             | [[Int]]?               | Порт (443, 8080)                          | кастомные порты           |
| `path`                             | String                 | Путь (/v1/users/123)                      | роутинг                   |
| `query`                            | String?                | Строка параметров (key=value&key2=value2) | ручной парсинг            |
| `queryItems` (через URLComponents) | [URLQueryItem]?        | Массив пар ключ-значение                  | безопасный доступ         |
| `fragment`                         | String?                | Якорь (#section)                          | навигация внутри страницы |
| `lastPathComponent`                | String                 | Последний сегмент пути (123)              | извлечение ID             |
| `pathExtension`                    | String                 | Расширение (.pdf, .jpg)                   | проверка типа файла       |
| `appendingPathComponent`           | URL                    | Добавляет сегмент к пути                  | построение путей          |
| `appendingQueryItems`              | URL (через components) | Добавляет параметры                       | динамические query        |

### 4. Полный разбор URL — продвинутый пример

```swift
func parseURL(_ string: String) {
    guard let url = URL(string: string) else {
        print("Некорректный URL")
        return
    }
    
    print("Полный URL:       ", url.absoluteString)
    print("Scheme:           ", url.scheme ?? "—")
    print("Host:             ", url.host ?? "—")
    print("Port:             ", url.port ?? "—")
    print("Path:             ", url.path)
    print("Last path component:", url.lastPathComponent)
    print("Query:            ", url.query ?? "—")
    print("Fragment:         ", url.fragment ?? "—")
    
    // Разбор query-параметров
    let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
    if let items = components?.queryItems {
        print("Параметры:")
        for item in items {
            print("  \(item.name): \(item.value ?? "—")")
        }
    }
}

// Тест
parseURL("https://api.example.com:8443/v2/products/456?sort=price-desc&limit=20&category=electronics#details")
```

Вывод:
```
Полный URL:        https://api.example.com:8443/v2/products/456?sort=price-desc&limit=20&category=electronics#details
Scheme:            https
Host:              api.example.com
Port:              8443
Path:              /v2/products/456
Last path component: 456
Query:             sort=price-desc&limit=20&category=electronics
Fragment:          details
Параметры:
  sort: price-desc
  limit: 20
  category: electronics
```

### 5. Реальные сценарии в iOS-разработке (2026)

#### Сценарий 1 — Загрузка данных из API (GET)

```swift
func fetchUser(id: String) async throws -> User {
    var components = URLComponents(string: "https://api.example.com/users")!
    components.path += "/\(id)"
    
    guard let url = components.url else { throw URLError(.badURL) }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
```

#### Сценарий 2 — Отправка данных (POST с query + body)

```swift
func searchUsers(query: String, page: Int = 1) async throws -> [User] {
    var components = URLComponents(string: "https://api.example.com/search/users")!
    components.queryItems = [
        URLQueryItem(name: "q", value: query),
        URLQueryItem(name: "page", value: "\(page)")
    ]
    
    guard let url = components.url else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode([User].self, from: data)
}
```

#### Сценарий 3 — Universal Links / Deep Linking

```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    
    // Разбор Universal Link
    let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
    let path = components?.path ?? ""
    let params = components?.queryItems ?? []
    
    switch path {
    case "/profile":
        if let id = params.first(where: { $0.name == "id" })?.value {
            openProfile(userID: id)
        }
    default:
        break
    }
}
```

#### Сценарий 4 — Работа с локальными файлами

```swift
func saveImage(_ image: UIImage, named filename: String) throws {
    guard let data = image.jpegData(compressionQuality: 0.8) else { return }
    
    let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
    let fileURL = documents.appendingPathComponent(filename)
    
    try data.write(to: fileURL, options: .atomic)
    print("Файл сохранён:", fileURL.path)
}
```

### 6. Таблица: самые полезные методы и свойства URL

| Метод / Свойство               | Что делает / возвращает                          | Когда использовать |
|--------------------------------|--------------------------------------------------|---------------------|
| `URL(string:)`                 | Создаёт URL из строки (опционал)                 | Почти всегда        |
| `URLComponents`                | Разбор и безопасное построение URL               | Динамические URL    |
| `appendingPathComponent`       | Добавляет сегмент к пути                         | Построение путей    |
| `appendingQueryItems`          | Добавляет параметры                              | Query-параметры     |
| `absoluteString`               | Полная строка                                    | Логи, отладка       |
| `scheme`, `host`, `path`       | Компоненты                                       | Роутинг, deep link  |
| `queryItems`                   | Массив пар ключ-значение                         | Парсинг параметров  |
| `lastPathComponent`            | Последний сегмент пути                           | Извлечение ID       |

### 7. Типичные ошибки и как их избежать

| Ошибка                          | Последствия                                 | Как избежать                                                     |
| ------------------------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| Конкатенация строк вручную      | Потеря /, ?, &, percent-encoding            | Используй URLComponents                                          |
| Force unwrap `URL(string:)`     | Crash при некорректном URL                  | Всегда [[if let]] / [[guard let]]                                |
| Нет проверки scheme == "https"  | Уязвимость, ATS-блокировка                  | Проверяй scheme                                                  |
| Неправильная обработка query    | Потеря параметров                           | Используй queryItems                                             |
| Забыли percent-encoding в query | Некорректные символы (пробелы, &, = и т.д.) | `addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)` |

### 8. Лучшие практики работы с URL в Swift 2026

- Никогда не конкатенируй строки вручную — только `URLComponents`
- Всегда проверяй `if let url = URL(string: …)` или `guard let`
- Для API — используй `HTTPS` + `Codable` + `async/await`
- Для deep linking — **Universal Links** по HTTPS + `userActivity.webpageURL`
- Для файлов — `FileManager.urls(for:in:)` + `appendingPathComponent`
- Для безопасности — проверяй `scheme == "https"` и `host` перед запросом
- Для отладки — логируй `url.absoluteString` и `url.queryItems`

**Короткий девиз**:
> «URL — это не строка. Это структурированный объект.  
> Строишь через URLComponents → парсишь через URL / URLComponents → работаешь безопасно и без багов.»
