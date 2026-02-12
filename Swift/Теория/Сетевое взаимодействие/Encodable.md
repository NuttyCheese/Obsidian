#network #Swift 
**Encodable** — это протокол, который позволяет типу **сериализовать** (encode) себя во внешний формат данных ([[JSON]], Property List, MessagePack, Protobuf, YAML и т.д.).

```swift
public protocol Encodable {
    func encode(to encoder: Encoder) throws
}
```

Он является частью **Codable** ([[Codable]] = `Encodable` & [[Decodable]]), но может использоваться отдельно, если вам нужно **только** сериализовать данные (например, отправить на сервер, сохранить в файл).

### Основные возможности Encodable

- Автоматическая генерация `encode(to:)` для большинства структур и классов
- Поддержка вложенных типов, опционалов, массивов, словарей
- Кастомизация через `CodingKeys`, `encode(to:)`, контейнеры
- Полная интеграция с **Swift Concurrency** ([[async]]/[[await]])
- Работает с любым `Encoder` ([[JSONEncoder]], PropertyListEncoder и т.д.)

### Базовый пример — автоматическая сериализация

```swift
struct User: Encodable {
    let id: UUID
    var name: String
    var age: Int
    var isActive: Bool
    var createdAt: Date
}

let user = User(
    id: UUID(),
    name: "Анна",
    age: 28,
    isActive: true,
    createdAt: Date()
)

// Сериализация в JSON
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
encoder.outputFormatting = .prettyPrinted

do {
    let data = try encoder.encode(user)
    let json = String(data: data, encoding: .utf8)!
    print(json)
    /*
    {
      "id" : "550E8400-E29B-41D4-A716-446655440000",
      "name" : "Анна",
      "age" : 28,
      "isActive" : true,
      "createdAt" : "2026-02-11T14:35:22Z"
    }
    */
} catch {
    print("Ошибка кодирования:", error)
}
```

### Продвинутые примеры

#### 1. Кастомные CodingKeys (snake_case в JSON)

```swift
struct Product: Encodable {
    let id: Int
    let title: String
    let priceInCents: Double
    
    enum CodingKeys: String, CodingKey {
        case id
        case title
        case priceInCents = "price_cents"  // ← меняем имя в JSON
    }
}
```

#### 2. Enum с associated values

```swift
enum NetworkResponse: Encodable {
    case success(data: Data)
    case failure(error: String)
    
    enum CodingKeys: String, CodingKey {
        case type, data, error
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        switch self {
        case .success(let data):
            try container.encode("success", forKey: .type)
            try container.encode(data, forKey: .data)
        case .failure(let error):
            try container.encode("failure", forKey: .type)
            try container.encode(error, forKey: .error)
        }
    }
}
```

#### 3. Кастомная стратегия кодирования дат

```swift
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .formatted({
    let formatter = DateFormatter()
    formatter.dateFormat = "dd.MM.yyyy HH:mm:ss"
    formatter.locale = Locale(identifier: "ru_RU")
    return formatter
}())
```

#### 4. Полиморфное кодирование (тип-дискриминатор)

```swift
protocol Animal: Encodable {
    var type: String { get }
}

struct Dog: Animal {
    let type = "dog"
    let name: String
    let breed: String
}

struct Cat: Animal {
    let type = "cat"
    let name: String
    let color: String
}

enum AnyAnimal: Encodable {
    case dog(Dog)
    case cat(Cat)
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        switch self {
        case .dog(let dog): try container.encode(dog)
        case .cat(let cat): try container.encode(cat)
        }
    }
}
```

#### 5. Отправка Encodable на сервер ([[async]]/[[await]])

```swift
struct CreateUserRequest: Encodable {
    let name: String
    let email: String
    let password: String
}

func createUser(_ request: CreateUserRequest) async throws -> User {
    let url = URL(string: "https://api.example.com/users")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let encoder = JSONEncoder()
    request.httpBody = try encoder.encode(request)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}

// Использование
Task {
    do {
        let request = CreateUserRequest(name: "Анна", email: "anna@example.com", password: "secret123")
        let newUser = try await createUser(request)
        print("Создан пользователь:", newUser.id)
    } catch {
        print("Ошибка:", error)
    }
}
```

#### 6. Encodable с keyEncodingStrategy

```swift
let encoder = JSONEncoder()
encoder.keyEncodingStrategy = .convertToSnakeCase  // camelCase → snake_case
```

### Лучшие практики Encodable (2026)

- Используй `CodingKeys` для несовпадения имён
- Для дат — кастомные стратегии (`formatted`, `secondsSince1970`, `iso8601`)
- Для полиморфизма — дискриминатор (`type`) + enum-wrapper
- Для больших данных — `.convertToSnakeCase` + `.compact` в продакшене
- Для ошибок — всегда обрабатывай `try` с `catch` (не `try!`)
- Для безопасности — валидируй данные после декодирования
- Для производительности — используй `JSONEncoder().outputFormatting = .compact`

**Короткое правило**:
> «Encodable — один из самых мощных инструментов Swift.  
> Делай модели `Encodable`, используй `CodingKeys`, кастомные стратегии и `async/await` — и отправка данных на сервер станет почти бесплатной.»
