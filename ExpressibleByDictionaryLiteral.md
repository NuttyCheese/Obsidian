**`ExpressibleByDictionaryLiteral`** — это протокол в стандартной библиотеке Swift, который позволяет типу быть инициализированным с помощью **литерала словаря** (dictionary literal), то есть синтаксиса `[key: value, key2: value2, …]`.

Если тип соответствует этому протоколу, вы можете писать:

```swift
let dict: MyCustomDict = ["name": "Alice", "age": 28, "active": true]
```

вместо более громоздкого создания через обычный `init`.

### Требования протокола (актуально на 2026 год)

```swift
public protocol ExpressibleByDictionaryLiteral {
    associatedtype Key   : Hashable
    associatedtype Value
    
    init(dictionaryLiteral elements: (Key, Value)...)
}
```

- `Key` должен быть `Hashable` (как ключи в обычном `Dictionary`)
- `Value` — любой тип
- Обязательный инициализатор принимает **variadic-параметр** пар `(Key, Value)`

### Типы из стандартной библиотеки, которые реализуют протокол

| Тип                              | Key                     | Value          | Пример литерала                              |
|----------------------------------|-------------------------|----------------|----------------------------------------------|
| `Dictionary<Key, Value>`         | `Key: Hashable`         | `Value`        | `["a": 1, "b": 2]`                           |
| `KeyValuePairs<Key, Value>`      | `Key`                   | `Value`        | `["one": 1, "two": 2]` (порядок сохраняется) |

### Самые популярные и полезные пользовательские реализации (2025–2026)

#### 1. Простой словарь с фиксированным набором ключей (самый частый учебный пример)

```swift
struct Config: ExpressibleByDictionaryLiteral {
    private var storage: [String: Any] = [:]
    
    init(dictionaryLiteral elements: (String, Any)...) {
        for (key, value) in elements {
            storage[key] = value
        }
    }
    
    subscript(key: String) -> Any? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
}

// Использование
let config: Config = [
    "apiKey": "abc123",
    "timeout": 30,
    "debug": true
]

print(config["apiKey"] as? String ?? "—")   // "abc123"
```

#### 2. Очень популярный паттерн — типобезопасный конфиг с известными ключами

```swift
enum ConfigKey: String, Hashable, CaseIterable {
    case apiBaseURL
    case maxRetries
    case enableLogging
}

struct AppConfig: ExpressibleByDictionaryLiteral {
    private var values: [ConfigKey: Any] = [:]
    
    init(dictionaryLiteral elements: (ConfigKey, Any)...) {
        for (key, value) in elements {
            values[key] = value
        }
    }
    
    subscript(_ key: ConfigKey) -> Any? {
        values[key]
    }
    
    var apiBaseURL: URL? {
        values[.apiBaseURL] as? URL
    }
    
    var maxRetries: Int? {
        values[.maxRetries] as? Int
    }
}

// Использование
let config: AppConfig = [
    .apiBaseURL: URL(string: "https://api.example.com")!,
    .maxRetries: 3,
    .enableLogging: true
]

print(config.apiBaseURL?.absoluteString ?? "—")   // "https://api.example.com"
```

#### 3. Словарь с валидацией при инициализации

```swift
struct Settings: ExpressibleByDictionaryLiteral {
    private(set) var theme: String = "system"
    private(set) var fontSize: Double = 16.0
    
    init(dictionaryLiteral elements: (String, Any)...) {
        for (key, value) in elements {
            switch key {
            case "theme":
                if let theme = value as? String, ["light", "dark", "system"].contains(theme) {
                    self.theme = theme
                }
            case "fontSize":
                if let size = value as? Double, size >= 12 && size <= 24 {
                    self.fontSize = size
                }
            default:
                break // игнорируем неизвестные ключи
            }
        }
    }
}

// Использование
let settings: Settings = [
    "theme": "dark",
    "fontSize": 18.0,
    "unknown": "ignored"
]

print(settings.theme)     // "dark"
print(settings.fontSize)  // 18.0
```

### Лучшие практики ExpressibleByDictionaryLiteral в Swift 2026

- **Используйте**, когда хотите дать типу **натуральный синтаксис** создания через `[key: value]`
- **Чаще всего** реализуют для:
  - конфигураций / настроек
  - словарей с фиксированным набором ключей
  - DSL (например, для валидации, стилей, параметров запросов)
  - обёрток над `[String: Any]` с типобезопасностью
- **Комбинируйте** с другими Expressible-протоколами:
  - `ExpressibleByStringLiteral`
  - `ExpressibleByIntegerLiteral`
  - `ExpressibleByArrayLiteral`
- **Не злоупотребляйте** — если тип требует сложной логики при создании → лучше обычный `init` с именованными параметрами
- **Swift 6 strict concurrency** — тип, реализующий протокол, должен быть `Sendable`, если используется в акторах
- **Документируйте** — пишите комментарий «ExpressibleByDictionaryLiteral — позволяет инициализацию через [ConfigKey: Any]»

**Короткий девиз 2026**:
> `ExpressibleByDictionaryLiteral` — это когда вы хотите, чтобы ваш тип создавался так же естественно, как обычный словарь: `["key": value]`.  
> В 2026 году:  
> - используйте для красивого API конфигураций, настроек, DSL  
> - чаще всего реализуют с фиксированным набором ключей (enum)  
> - комбинируйте с другими Expressible*-протоколами  
> Это **мощный инструмент** для declarative и выразительного кода.

Удачи с элегантными инициализациями словарей в твоём проекте! 📖