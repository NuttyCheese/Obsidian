**Codable** — это протокол, который позволяет легко **сериализовать** (encode) и **десериализовать** (decode) типы данных в/из внешних представлений ([[JSON]], Property List, MessagePack, Protobuf и т.д.).

`Codable` = [[Encodable]] + [[Decodable]]

```swift
public typealias Codable = Decodable & Encodable
```

### Основные возможности и преимущества

- Автоматическая генерация `encode(to:)` и `init(from:)` для большинства структур/классов
- Поддержка вложенных типов, опционалов, массивов, словарей
- Кастомизация через `CodingKeys`, `encode(to:)`, `init(from:)`
- Работает с любым `Encoder`/`Decoder` (JSON, PropertyList, MessagePack и т.д.)
- Полная интеграция с **Swift Concurrency** ([[async]]/[[await]])
- Поддержка **[[generic]]**, **[[AssociatedType]]**, **[[enum]] with associated values**

### Базовый пример — простой [[struct]]

```swift
struct User: Codable {
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

// Encoding → JSON
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
encoder.outputFormatting = .prettyPrinted

if let data = try? encoder.encode(user) {
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
}

// Decoding ← JSON
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

if let decoded = try? decoder.decode(User.self, from: data) {
    print(decoded.name) // Анна
}
```

### Продвинутые примеры

#### 1. Кастомные CodingKeys (разные имена в JSON)

```swift
struct Product: Codable {
    let id: Int
    let title: String
    let priceInCents: Double
    
    enum CodingKeys: String, CodingKey {
        case id
        case title
        case priceInCents = "price_cents"  // ← snake_case в JSON
    }
}
```

#### 2. Enum с associated values

```swift
enum NetworkResponse: Codable {
    case success(data: Data)
    case failure(error: String)
    
    enum CodingKeys: String, CodingKey {
        case type, data, error
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "success":
            let data = try container.decode(Data.self, forKey: .data)
            self = .success(data: data)
        case "failure":
            let error = try container.decode(String.self, forKey: .error)
            self = .failure(error: error)
        default:
            throw DecodingError.dataCorruptedError(forKey: .type, in: container, debugDescription: "Invalid type")
        }
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

#### 3. Date с кастомной стратегией

```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .formatted({
    let formatter = DateFormatter()
    formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
    formatter.locale = Locale(identifier: "en_US_POSIX")
    return formatter
}())

let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
```

#### 4. Полиморфизм и Codable (тип-дискриминатор)

```swift
protocol Animal: Codable {
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

enum AnimalWrapper: Codable {
    case dog(Dog)
    case cat(Cat)
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        let type = try container.decode(String.self)
        
        switch type {
        case "dog": self = .dog(try container.decode(Dog.self))
        case "cat": self = .cat(try container.decode(Cat.self))
        default: throw DecodingError.dataCorruptedError(in: container, debugDescription: "Unknown animal type")
        }
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        switch self {
        case .dog(let dog): try container.encode(dog)
        case .cat(let cat): try container.encode(cat)
        }
    }
}
```

#### 5. [[async]] [[JSON]] decoding (Swift 5.5+)

```swift
struct Post: Codable {
    let id: Int
    let title: String
}

func fetchPosts() async throws -> [Post] {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Post].self, from: data)
}

// Использование
Task {
    do {
        let posts = try await fetchPosts()
        print(posts.count) // 100
    } catch {
        print("Error:", error)
    }
}
```

### Лучшие практики Codable (2026)

- Используй `CodingKeys` для snake_case / camelCase различий
- Для дат — кастомные стратегии (`formatted`, `secondsSince1970`, `iso8601`)
- Для полиморфизма — дискриминатор (`type`) + enum-wrapper
- Для больших JSON — используй `JSONDecoder().keyDecodingStrategy = .convertFromSnakeCase`
- Для производительности — `JSONEncoder().outputFormatting = .compact` в продакшене
- Избегай `try!` в production — всегда обрабатывай ошибки

**Короткое правило**:
> «Codable — один из самых мощных и удобных инструментов Swift.  
> Делай модели `Codable`, используй `CodingKeys`, кастомные стратегии и `async`/`await` — и работа с сетью станет почти бесплатной.»
