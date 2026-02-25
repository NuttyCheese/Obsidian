**`URLQueryItem`** — это простая структура в [[Foundation]], представляющая **одну пару ключ-значение** в строке запроса [[URL]] (query string).

Она используется исключительно внутри [[URLComponent]] для **безопасного** формирования и парсинга параметров запроса (`?key1=value1&key2=value2`).

### 1. Почему URLQueryItem лучше ручного форматирования query-строки

| Проблема при ручном форматировании | Последствия | Как решает URLQueryItem |
|------------------------------------|-------------|--------------------------|
| Пробелы, &, =, ?, # в значениях    | Сломанный URL | Автоматическое percent-encoding |
| Дубликаты ключей                   | Непредсказуемое поведение | Поддерживает несколько значений для одного ключа |
| Забыли экранировать                | Сервер не поймёт параметры | Всё экранируется правильно |
| Сложный парсинг строки             | Ошибки при разборе | `URLComponents.queryItems` → готовый массив |

### 2. Структура URLQueryItem

```swift
public struct URLQueryItem : Hashable, Sendable {
    public var name: String
    public var value: String?
    
    public init(name: String, value: String?)
}
```

- `name` — ключ (обязателен)
- `value` — значение (может быть [[nil]])

### 3. Все возможные сценарии использования

#### 3.1 Создание и добавление query-параметров

```swift
var components = URLComponents(string: "https://api.example.com/search")!

// Вариант 1 — массив сразу
components.queryItems = [
    URLQueryItem(name: "q", value: "Swift programming"),
    URLQueryItem(name: "page", value: "1"),
    URLQueryItem(name: "limit", value: "20")
]

// Вариант 2 — добавление по одному
components.queryItems = []
components.queryItems?.append(URLQueryItem(name: "category", value: "books"))
components.queryItems?.append(URLQueryItem(name: "sort", value: "price-desc"))

// Вариант 3 — добавление nil-значения (ключ без значения)
components.queryItems?.append(URLQueryItem(name: "premium", value: nil))
// → ?premium

print(components.url?.absoluteString ?? "")
// https://api.example.com/search?q=Swift+programming&page=1&limit=20&category=books&sort=price-desc&premium
```

#### 3.2 Парсинг query-параметров из любого URL

```swift
let urlString = "https://example.com/search?q=Swift+programming&page=2&filter=free&sort=recent&premium"

if let url = URL(string: urlString),
   let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
   let queryItems = components.queryItems {
    
    print("Все параметры:")
    for item in queryItems {
        print("  \(item.name): \(item.value ?? "<nil>")")
    }
    
    // Поиск конкретного параметра
    if let page = queryItems.first(where: { $0.name == "page" })?.value {
        print("Страница:", page) // Страница: 2
    }
    
    // Все значения для ключа (если дубликаты)
    let filters = queryItems.filter { $0.name == "filter" }.compactMap { $0.value }
    print("Фильтры:", filters)
}
```

#### 3.3 Работа с несколькими значениями одного ключа

```swift
var components = URLComponents(string: "https://example.com")!
components.queryItems = [
    URLQueryItem(name: "tag", value: "swift"),
    URLQueryItem(name: "tag", value: "ios"),
    URLQueryItem(name: "tag", value: "mobile")
]

print(components.url!.absoluteString)
// https://example.com?tag=swift&tag=ios&tag=mobile
```

#### 3.4 Обновление / удаление параметров

```swift
var components = URLComponents(string: "https://example.com/search?q=swift&page=1&sort=asc")!

// Удаление параметра
components.queryItems?.removeAll { $0.name == "sort" }

// Изменение значения
if let index = components.queryItems?.firstIndex(where: { $0.name == "page" }) {
    components.queryItems?[index] = URLQueryItem(name: "page", value: "3")
}

// Добавление нового
components.queryItems?.append(URLQueryItem(name: "filter", value: "premium"))

print(components.url!.absoluteString)
// https://example.com/search?q=swift&page=3&filter=premium
```

#### 3.5 Получение всех значений для ключа (включая дубликаты)

```swift
extension URLComponents {
    func values(for name: String) -> [String] {
        queryItems?
            .filter { $0.name == name }
            .compactMap { $0.value } ?? []
    }
}

let url = URL(string: "https://example.com?tag=swift&tag=ios&tag=swiftui")!
if let components = URLComponents(url: url, resolvingAgainstBaseURL: false) {
    let tags = components.values(for: "tag")
    print(tags) // ["swift", "ios", "swiftui"]
}
```

### 4. Реальные сценарии в iOS-разработке (2026)

#### Сценарий 1 — Поиск с пагинацией и фильтрами

```swift
func buildSearchURL(query: String, page: Int, filters: [String: String]) -> URL? {
    var components = URLComponents(string: "https://api.example.com/search")!
    
    components.queryItems = [
        URLQueryItem(name: "q", value: query),
        URLQueryItem(name: "page", value: "\(page)")
    ]
    
    for (key, value) in filters {
        components.queryItems?.append(URLQueryItem(name: key, value: value))
    }
    
    return components.url
}
```

#### Сценарий 2 — [[Universal Link]] с параметрами

```swift
func handleUniversalLink(url: URL) {
    guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false) else { return }
    
    let path = components.path
    let params = components.queryItems ?? []
    
    switch path {
    case "/product":
        if let id = params.first(where: { $0.name == "id" })?.value,
           let category = params.first(where: { $0.name == "category" })?.value {
            openProduct(id: id, category: category)
        }
    default:
        break
    }
}
```

#### Сценарий 3 — [[API]]-запрос с динамическими параметрами

```swift
func fetchItems(category: String?, sort: String?, page: Int) async throws -> [Item] {
    var components = URLComponents(string: "https://api.example.com/items")!
    
    if let category = category {
        components.queryItems?.append(URLQueryItem(name: "category", value: category))
    }
    if let sort = sort {
        components.queryItems?.append(URLQueryItem(name: "sort", value: sort))
    }
    components.queryItems?.append(URLQueryItem(name: "page", value: "\(page)"))
    
    guard let url = components.url else { throw URLError(.badURL) }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Item].self, from: data)
}
```

### 5. Таблица: самые полезные методы и свойства

| Метод / Свойство               | Что делает / возвращает                          | Когда использовать |
|--------------------------------|--------------------------------------------------|---------------------|
| `URLComponents()`              | Пустой компонент                                 | Создание с нуля     |
| `URLComponents(string:)`       | Парсит готовый URL                               | Разбор строки       |
| `URLComponents(url:resolvingAgainstBaseURL:)` | Парсит с базовым URL                  | Relative URL        |
| `url`                          | Готовый URL                                      | Передача в запрос   |
| `queryItems`                   | Массив `URLQueryItem`                            | Доступ к параметрам |
| `queryItems?.append(...)`      | Добавляет параметр                               | Динамическое добавление |
| `queryItems?.removeAll(...)`   | Удаляет параметры                                | Фильтрация          |

### 6. Типичные ошибки и как их избежать

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Ручная конкатенация query-строки            | Сломанный URL, потеря параметров             | Только `URLComponents` + `queryItems` |
| Force unwrap `components.url`               | Crash при ошибке                             | `if let url = components.url` |
| Забыли percent-encoding вручную             | Некорректные символы                         | `URLQueryItem` делает автоматически |
| Дубликаты ключей без обработки              | Сервер может взять только первое значение    | Обрабатывать через `filter` или массив значений |
| Неправильный парсинг (работа с `query` вместо `queryItems`) | Сложный и ошибочный код                 | Всегда `queryItems` |

### 7. Лучшие практики 2026 года

- Никогда не строй query-строку вручную — только `URLQueryItem`
- Всегда проверяй `if let url = components.url` или `guard let`
- Для [[API]] — **[[HTTPS]]** + [[Codable]] + [[async]]/[[await]]
- Для deep linking — **Universal Links** по HTTPS + `URLComponents`
- Для динамических параметров — используй `URLQueryItem` и избегай дубликатов
- Для отладки — логируй `components.url?.absoluteString`
- Для больших query — используй `URLQueryItem` и проверяй длину URL (< 2000–8000 символов в зависимости от сервера)

**Короткий девиз**:
> «URLComponents + URLQueryItem — единственный безопасный способ строить и разбирать URL с параметрами в Swift.  
> Забудь про ручную конкатенацию — это источник багов, уязвимостей и головной боли.»
