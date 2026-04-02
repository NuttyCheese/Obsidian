#swift #rawrepresentable #protocol #enum #raw-value #codable

---
### Определение
**`RawRepresentable`** — это протокол в [[Swift]], который позволяет типу быть представленным через некоторое "сырое" значение (raw value) другого типа. Этот протокол связывает экземпляр типа с его простым представлением (например, строкой, целым числом или любым другим типом), которое может быть легко сохранено, передано или сериализовано.

Протокол требует реализации двух методов:
- **`init?(rawValue: RawValue)`** — инициализатор, создающий экземпляр из сырого значения (может вернуть `nil`, если значение некорректно).
- **`var rawValue: RawValue { get }`** — свойство, возвращающее сырое представление экземпляра.

### Зачем это знать iOS-разработчику?
1.  **Сырые значения перечислений ([[enum]]s):** Все перечисления с `rawValue` автоматически соответствуют `RawRepresentable`.
2.  **Сериализация и десериализация:** Удобное преобразование между пользовательскими типами и примитивами ([[String]], [[Int]], [[Data]]).
3.  **[[UserDefaults]]:** Хранение кастомных типов через raw значения.
4.  **[[URL]] и параметры запросов:** Преобразование enum-кейсов в строки для передачи в [[API]].
5.  **Создание типобезопасных оберток:** Например, `struct UserID: RawRepresentable` с `rawValue: Int`.

---

### Базовый синтаксис

```swift
protocol RawRepresentable<RawValue> {
    associatedtype RawValue
    init?(rawValue: RawValue)
    var rawValue: RawValue { get }
}
```

#### Пример с перечислением (автоматическая реализация)

```swift
enum Direction: String {
    case north = "N"
    case south = "S"
    case east = "E"
    case west = "W"
}

// Swift автоматически добавляет соответствие RawRepresentable
let direction = Direction(rawValue: "N")   // .north
print(direction?.rawValue ?? "")           // "N"
```

#### Пример со структурой (ручная реализация)

```swift
struct UserRole: RawRepresentable {
    let rawValue: String
    
    static let admin = UserRole(rawValue: "admin")
    static let editor = UserRole(rawValue: "editor")
    static let viewer = UserRole(rawValue: "viewer")
    
    init?(rawValue: String) {
        switch rawValue {
        case "admin", "editor", "viewer":
            self.rawValue = rawValue
        default:
            return nil
        }
    }
}

let role = UserRole(rawValue: "admin")
print(role?.rawValue ?? "")  // admin
```

---

### Перечисления с raw значениями

Swift автоматически синтезирует `RawRepresentable` для `enum` с указанием типа raw value.

#### Целочисленные raw значения

```swift
enum HttpStatus: Int {
    case ok = 200
    case notFound = 404
    case internalError = 500
}

let status = HttpStatus(rawValue: 404)  // .notFound
print(status?.rawValue ?? 0)            // 404
```

#### Строковые raw значения

```swift
enum Icon: String {
    case home = "house.fill"
    case search = "magnifyingglass"
    case profile = "person.circle"
}

let icon = Icon(rawValue: "house.fill")  // .home
print(icon?.rawValue ?? "")              // "house.fill"
```

#### [[Float]] / [[Double]] raw значения

```swift
enum Version: Double {
    case v1 = 1.0
    case v2 = 1.5
    case v3 = 2.0
}

let version = Version(rawValue: 1.5)  // .v2
print(version?.rawValue ?? 0)         // 1.5
```

---

### Структуры и `RawRepresentable`

#### ID-обертки (типобезопасные идентификаторы)

```swift
struct UserID: RawRepresentable, Equatable {
    let rawValue: Int
    
    init?(rawValue: Int) {
        guard rawValue > 0 else { return nil }
        self.rawValue = rawValue
    }
}

struct ProductID: RawRepresentable, Equatable {
    let rawValue: String
    
    init?(rawValue: String) {
        guard !rawValue.isEmpty else { return nil }
        self.rawValue = rawValue
    }
}

let userID = UserID(rawValue: 42)
let productID = ProductID(rawValue: "ABC-123")
```

#### Настройки приложения

```swift
struct AppTheme: RawRepresentable {
    let rawValue: String
    
    static let light = AppTheme(rawValue: "light")
    static let dark = AppTheme(rawValue: "dark")
    static let system = AppTheme(rawValue: "system")
    
    init?(rawValue: String) {
        switch rawValue {
        case "light", "dark", "system":
            self.rawValue = rawValue
        default:
            return nil
        }
    }
}

// UserDefaults
UserDefaults.standard.set(AppTheme.dark.rawValue, forKey: "selectedTheme")

if let themeRaw = UserDefaults.standard.string(forKey: "selectedTheme"),
   let theme = AppTheme(rawValue: themeRaw) {
    applyTheme(theme)
}
```

---

### RawRepresentable и Codable

Типы, соответствующие `RawRepresentable`, автоматически поддерживают `Codable`, если `RawValue` тоже `Codable`.

```swift
enum Currency: String, Codable {
    case usd = "USD"
    case eur = "EUR"
    case rub = "RUB"
}

let currency = Currency.eur
let encoded = try JSONEncoder().encode(currency)
let decoded = try JSONDecoder().decode(Currency.self, from: encoded)
print(decoded)  // eur
```

#### Кастомная структура с RawRepresentable и [[Codable]]

```swift
struct CountryCode: RawRepresentable, Codable {
    let rawValue: String
    
    static let usa = CountryCode(rawValue: "US")
    static let canada = CountryCode(rawValue: "CA")
    static let uk = CountryCode(rawValue: "UK")
    
    init?(rawValue: String) {
        switch rawValue {
        case "US", "CA", "UK":
            self.rawValue = rawValue
        default:
            return nil
        }
    }
}

let country = CountryCode.usa
let data = try JSONEncoder().encode(country)
let decodedCountry = try JSONDecoder().decode(CountryCode.self, from: data)
print(decodedCountry.rawValue)  // US
```

---

### RawRepresentable и [[CaseIterable]]

Комбинация позволяет перебирать все возможные raw значения.

```swift
enum SortOption: String, CaseIterable {
    case name = "По имени"
    case date = "По дате"
    case price = "По цене"
}

let allRawValues = SortOption.allCases.map { $0.rawValue }
print(allRawValues)  // ["По имени", "По дате", "По цене"]
```

---

### RawRepresentable с ассоциированными значениями

Если перечисление имеет ассоциированные значения, `RawRepresentable` не синтезируется автоматически, но можно реализовать вручную.

```swift
enum NetworkResponse {
    case success(data: Data)
    case failure(code: Int, message: String)
}

extension NetworkResponse: RawRepresentable {
    typealias RawValue = String
    
    init?(rawValue: String) {
        // Парсинг строки
        if rawValue.hasPrefix("success:") {
            let dataString = String(rawValue.dropFirst(8))
            guard let data = dataString.data(using: .utf8) else { return nil }
            self = .success(data: data)
        } else if rawValue.hasPrefix("failure:") {
            let components = rawValue.dropFirst(8).split(separator: "|")
            guard components.count == 2,
                  let code = Int(components[0]) else { return nil }
            self = .failure(code: code, message: String(components[1]))
        } else {
            return nil
        }
    }
    
    var rawValue: String {
        switch self {
        case .success(let data):
            return "success:\(String(data: data, encoding: .utf8) ?? "")"
        case .failure(let code, let message):
            return "failure:\(code)|\(message)"
        }
    }
}
```

---

### RawRepresentable и UserDefaults

Хранение кастомных типов в `UserDefaults` через raw значения.

```swift
extension UserDefaults {
    func set<T: RawRepresentable>(_ value: T?, forKey key: String) where T.RawValue == String {
        set(value?.rawValue, forKey: key)
    }
    
    func value<T: RawRepresentable>(forKey key: String) -> T? where T.RawValue == String {
        guard let raw = string(forKey: key) else { return nil }
        return T(rawValue: raw)
    }
}

// Использование
enum Language: String {
    case english = "en"
    case russian = "ru"
}

UserDefaults.standard.set(Language.english, forKey: "appLanguage")
let lang: Language? = UserDefaults.standard.value(forKey: "appLanguage")
print(lang ?? "")  // english
```

---

### RawRepresentable и [[URL]]

```swift
struct APIEndpoint: RawRepresentable {
    let rawValue: String
    
    static let users = APIEndpoint(rawValue: "/users")
    static let posts = APIEndpoint(rawValue: "/posts")
    static let comments = APIEndpoint(rawValue: "/comments")
    
    var url: URL? {
        return URL(string: "https://api.example.com" + rawValue)
    }
}

let endpoint = APIEndpoint.users
print(endpoint.url?.absoluteString ?? "")  // https://api.example.com/users
```

---

### Лучшие практики

#### 1. **Используйте для конечных наборов значений**

```swift
// ✅ Хорошо — enum с rawValue
enum Status: String {
    case active, inactive, pending
}

// ❌ Плохо — для динамических данных
struct DynamicData: RawRepresentable {
    // rawValue не имеет смысла
}
```

#### 2. **Всегда проверяйте валидность в `init?(rawValue:)`**

```swift
struct Email: RawRepresentable {
    let rawValue: String
    
    init?(rawValue: String) {
        // Валидация email
        let regex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES %@", regex)
        guard predicate.evaluate(with: rawValue) else { return nil }
        self.rawValue = rawValue
    }
}
```

#### 3. **Используйте `RawRepresentable` для типобезопасных оберток**

```swift
struct UserID: RawRepresentable, Hashable {
    let rawValue: Int
}

struct SessionToken: RawRepresentable, Hashable {
    let rawValue: String
}
```

#### 4. **Комбинируйте с `Codable` для сериализации**

```swift
enum Priority: String, Codable {
    case low, medium, high
}

let priority = Priority.medium
let data = try JSONEncoder().encode(priority)
```

---

### Ограничения

1.  **Не работает с ассоциированными значениями** без ручной реализации.
2.  **RawValue должен быть конкретным типом** (не протоколом).
3.  **Инициализатор может вернуть `nil`** — нужно обрабатывать опциональность.

---

### Короткое правило

> **`RawRepresentable`** связывает тип с его простым представлением.  
> Перечисления с `rawValue` реализуют его автоматически.  
> Используйте для сериализации, хранения в `UserDefaults` и типобезопасных идентификаторов.

### Итог

**`RawRepresentable`** в Swift:

1.  **Позволяет преобразовывать тип в примитив** (String, Int, Data и т.д.) и обратно.
2.  **Автоматически синтезируется для `enum`** с указанием типа raw value.
3.  **Может быть реализован вручную** для [[struct]] и [[enum]] с ассоциированными значениями.
4.  **Используется в `UserDefaults`, `Codable`, параметрах запросов**.
5.  **Дает типобезопасность** при работе с сырыми значениями.

Понимание `RawRepresentable` помогает создавать чистые, типобезопасные API и упрощает работу с сериализацией и хранением данных .