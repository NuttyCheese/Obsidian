**URLComponents** — это структура в [[Foundation]], предназначенная для **безопасного парсинга, создания и модификации** компонентов [[URL]].

В отличие от `URL(string:)`, который может вернуть [[nil]] при любой ошибке, `URLComponents` позволяет:

- строить URL по частям (scheme, host, path, queryItems и т.д.)  
- автоматически правильно экранировать query-параметры  
- безопасно добавлять/удалять/изменять части URL  
- разбирать любой существующий URL на компоненты  

Это **единственный рекомендуемый** способ динамически создавать или изменять URL в Swift.

### 1. Почему URLComponents лучше, чем ручная конкатенация строк

| Проблема при ручной конкатенации | Последствия | Как решает URLComponents |
|-----------------------------------|-------------|---------------------------|
| Забыли `/` между путями           | Некорректный URL | Автоматически добавляет `/` |
| Пробелы, &, = в query             | Сломанный запрос | Автоматическое percent-encoding |
| Неправильный порядок параметров   | Сервер не поймёт | queryItems — массив пар |
| Дубликаты ключей                  | Непредсказуемое поведение | Поддерживает несколько значений для одного ключа |
| Ошибка в scheme/host/port         | Crash или неверный запрос | Каждое свойство проверяется |

### 2. Основные свойства URLComponents

| Свойство              | Тип               | Что хранит / возвращает                     | Изменяемое? | Пример значения              |
| --------------------- | ----------------- | ------------------------------------------- | ----------- | ---------------------------- |
| `scheme`              | [[String]]?       | Протокол ([[HTTPS]], [[HTTP]], file, myapp) | Да          | "https"                      |
| `user`                | `String?`         | Имя пользователя (deprecated)               | Да          | —                            |
| `password`            | `String?`         | Пароль (deprecated)                         | Да          | —                            |
| `host`                | `String?`         | Домен или IP (api.example.com)              | Да          | "api.example.com"            |
| `port`                | [[Int]]?          | Порт (443, 8080)                            | Да          | 443                          |
| `path`                | `String`          | Путь (/v1/users/123)                        | Да          | "/v1/users"                  |
| `query`               | `String?`         | Строка параметров (key=value&key2=value2)   | Да          | "page=2&sort=desc"           |
| `fragment`            | `String?`         | Якорь (#section1)                           | Да          | "section1"                   |
| `queryItems`          | [[URLQueryItem]]? | Массив пар ключ-значение                    | Да          | [{name: "page", value: "2"}] |
| `percentEncodedQuery` | `String?`         | То же, что query, но уже закодировано       | Да          | —                            |
| `url`                 | `URL?`            | Готовый объект URL                          | Нет         | —                            |
| `string`              | `String?`         | Полная строка URL                           | Нет         | —                            |

### 3. Создание и модификация URL — все варианты

#### 3.1 Простейшее создание

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
    print(url.absoluteString)
    // https://api.example.com/v1/users?sort=name&limit=50
}
```

#### 3.2 Парсинг существующего URL

```swift
if let url = URL(string: "https://api.example.com/v2/products/456?category=electronics&sort=price-desc&limit=20#details"),
   var components = URLComponents(url: url, resolvingAgainstBaseURL: false) {
    
    print("Scheme:    ", components.scheme ?? "—")          // https
    print("Host:      ", components.host ?? "—")            // api.example.com
    print("Path:      ", components.path)                   // /v2/products/456
    print("Query:     ", components.query ?? "—")           // category=electronics&sort=price-desc&limit=20
    print("Fragment:  ", components.fragment ?? "—")        // details
    
    // Работа с queryItems
    components.queryItems?.forEach { item in
        print("\(item.name): \(item.value ?? "—")")
    }
    // category: electronics
    // sort: price-desc
    // limit: 20
    
    // Модификация
    components.queryItems?.append(URLQueryItem(name: "page", value: "2"))
    if let newURL = components.url {
        print("Новый URL:", newURL)
        // https://api.example.com/v2/products/456?category=electronics&sort=price-desc&limit=20&page=2#details
    }
}
```

#### 3.3 Добавление/изменение пути и параметров

```swift
var components = URLComponents(string: "https://example.com/api")!
components.path += "/v1/users/123"
components.queryItems = [
    URLQueryItem(name: "expand", value: "profile,posts"),
    URLQueryItem(name: "fields", value: "name,email")
]

print(components.url?.absoluteString ?? "")
// https://example.com/api/v1/users/123?expand=profile,posts&fields=name,email
```

#### 3.4 Работа с портом, fragment и userinfo (редко, но важно)

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "api.example.com"
components.port = 8443
components.user = "admin"
components.password = "secret123"
components.path = "/admin"
components.fragment = "logs"

print(components.url?.absoluteString ?? "")
// https://admin:secret123@api.example.com:8443/admin#logs
```

### 4. Реальные сценарии в [[iOS]]-разработке (2026)

#### Сценарий 1 — Поиск с динамическими параметрами

```swift
func searchProducts(query: String, category: String?, page: Int = 1) async throws -> [Product] {
    var components = URLComponents(string: "https://api.example.com/products")!
    components.queryItems = [
        URLQueryItem(name: "q", value: query),
        URLQueryItem(name: "page", value: "\(page)"),
        URLQueryItem(name: "limit", value: "20")
    ]
    
    if let category = category {
        components.queryItems?.append(URLQueryItem(name: "category", value: category))
    }
    
    guard let url = components.url else { throw URLError(.badURL) }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Product].self, from: data)
}
```

#### Сценарий 2 — [[Universal Link]] / [[deep link]]

```swift
func handleUniversalLink(url: URL) {
    guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false) else { return }
    
    let path = components.path
    let params = components.queryItems ?? []
    
    switch path {
    case "/profile":
        if let id = params.first(where: { $0.name == "id" })?.value {
            openProfile(userID: id)
        }
    case "/product":
        if let id = params.first(where: { $0.name == "id" })?.value {
            openProduct(productID: id)
        }
    default:
        break
    }
}
```

#### Сценарий 3 — Построение [[API]]-запроса с пагинацией

```swift
func fetchUsers(page: Int, limit: Int = 20, search: String? = nil) async throws -> [User] {
    var components = URLComponents(string: "https://api.example.com/v2/users")!
    components.queryItems = [
        URLQueryItem(name: "page", value: "\(page)"),
        URLQueryItem(name: "limit", value: "\(limit)")
    ]
    
    if let search = search, !search.isEmpty {
        components.queryItems?.append(URLQueryItem(name: "search", value: search))
    }
    
    guard let url = components.url else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode([User].self, from: data)
}
```

### 5. Таблица: самые полезные методы и свойства

| Метод / Свойство                              | Что делает / возвращает          | Когда использовать             |
| --------------------------------------------- | -------------------------------- | ------------------------------ |
| `URLComponents()`                             | Пустой компонент                 | Создание с нуля                |
| `URLComponents(string:)`                      | Парсит готовый URL               | Разбор существующего           |
| `URLComponents(url:resolvingAgainstBaseURL:)` | Парсит с учётом базового URL     | Relative URL                   |
| `url`                                         | Готовый URL (опционал)           | Передача в [[URLRequest]]      |
| `scheme`, `host`, `path`                      | Основные компоненты              | Роутинг, deep link             |
| `queryItems`                                  | Массив `URLQueryItem`            | Безопасный доступ к параметрам |
| `percentEncodedQuery`                         | Закодированная строка параметров | Отладка                        |
| `queryItems?.append(...)`                     | Добавляет параметр               | Динамические query             |

### 6. Типичные ошибки и как их избежать

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Ручная конкатенация строк                   | Сломанный URL, потеря параметров             | Только URLComponents |
| Force unwrap `components.url`               | Crash при ошибке                             | `if let url = components.url` |
| Нет percent-encoding в query                | Некорректные символы (пробелы, &, =)         | URLComponents делает автоматически |
| Забыли scheme или host                      | Некорректный URL                             | Всегда проверять |
| Использование `query` вместо `queryItems`   | Сложный парсинг                              | Всегда queryItems |

### 7. Лучшие практики 2026 года

- Никогда не строй URL через `+` — только `URLComponents`
- Всегда проверяй `if let url = components.url` или `guard let`
- Для API — **HTTPS** + `Codable` + `async/await`
- Для deep linking — **Universal Links** по HTTPS + `URLComponents(url: ...)`
- Для файлов — `FileManager.urls(for:in:)` + `appending(path:)`
- Для безопасности — проверяй `scheme == "https"` и `host`
- Для отладки — логируй `components.url?.absoluteString`
- Для больших query — используй `URLQueryItem` и избегай дубликатов ключей

**Короткий девиз**:
> «URLComponents — это единственный безопасный способ строить и разбирать URL в Swift.  
> Забудь про ручную конкатенацию строк — это источник багов и уязвимостей.»
