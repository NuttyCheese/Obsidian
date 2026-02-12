#network #Swift 
**[[Decodable]]** — это протокол, который позволяет типу инициализироваться из внешнего представления данных ([[JSON]], Property List, MessagePack, Protobuf и т.д.).

```swift
public protocol Decodable {
    init(from decoder: Decoder) throws
}
```

Он является частью **[[Codable]]** (`Codable = Decodable &` [[Encodable]]), но может использоваться отдельно, если вам нужно только десериализовать данные.

### Основные возможности Decodable

- Автоматическая генерация `init(from:)` для большинства структур и классов
- Поддержка вложенных типов, опционалов, массивов, словарей
- Кастомизация через `CodingKeys`, `init(from:)`, контейнеры
- Полная интеграция с **Swift Concurrency** (`async/await`)
- Работает с любым `Decoder` ([[JSONDecoder]], PropertyListDecoder и т.д.)

### Базовый пример — автоматическое декодирование

```swift
struct User: Decodable {
    let id: UUID
    let name: String
    let age: Int
    let isActive: Bool
    let createdAt: Date
}

let json = """
{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Анна",
    "age": 28,
    "isActive": true,
    "createdAt": "2026-02-11T14:35:22Z"
}
"""

if let data = json.data(using: .utf8) {
    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    
    do {
        let user = try decoder.decode(User.self, from: data)
        print("User: \(user.name), возраст: \(user.age)")
    } catch {
        print("Ошибка декодирования:", error.localizedDescription)
    }
}
```

### Продвинутые примеры

#### 1. Кастомные CodingKeys (snake_case → camelCase)

```swift
struct Product: Decodable {
    let id: Int
    let title: String
    let priceInCents: Double
    
    enum CodingKeys: String, CodingKey {
        case id
        case title
        case priceInCents = "price_cents"
    }
}
```

#### 2. Enum с associated values

```swift
enum NetworkResponse: Decodable {
    case success(data: Data)
    case failure(error: String)
    
    private enum CodingKeys: String, CodingKey {
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
            throw DecodingError.dataCorruptedError(forKey: .type, in: container, debugDescription: "Unknown type")
        }
    }
}
```

#### 3. Полиморфное декодирование (тип-дискриминатор)

```swift
protocol Animal: Decodable {
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

enum AnyAnimal: Decodable {
    case dog(Dog)
    case cat(Cat)
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        let typeContainer = try decoder.container(keyedBy: CodingKeys.self)
        let type = try typeContainer.decode(String.self, forKey: .type)
        
        switch type {
        case "dog": self = .dog(try container.decode(Dog.self))
        case "cat": self = .cat(try container.decode(Cat.self))
        default:
            throw DecodingError.dataCorruptedError(in: container, debugDescription: "Unknown animal type")
        }
    }
    
    private enum CodingKeys: String, CodingKey {
        case type
    }
}
```

#### 4. Декодирование с [[async]]/[[await]] (реальный [[API]]-запрос)

```swift
struct Post: Decodable {
    let id: Int
    let title: String
    let body: String
}

func fetchPosts() async throws -> [Post] {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
    let (data, _) = try await URLSession.shared.data(from: url)
    
    let decoder = JSONDecoder()
    return try decoder.decode([Post].self, from: data)
}

// Использование
Task {
    do {
        let posts = try await fetchPosts()
        print("Получено постов: \(posts.count)")
    } catch {
        print("Ошибка:", error.localizedDescription)
    }
}
```

#### 5. Кастомная стратегия декодирования дат

```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .formatted({
    let formatter = DateFormatter()
    formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
    formatter.locale = Locale(identifier: "en_US_POSIX")
    return formatter
}())
```

#### 6. Декодирование с keyDecodingStrategy

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase  // автоматически snake_case → camelCase
```

### Лучшие практики Codable / Decodable (2026)

- Используй `CodingKeys` для несовпадения имён
- Для дат — кастомные стратегии (`formatted`, `secondsSince1970`, `iso8601`)
- Для полиморфизма — дискриминатор (`type`) + [[enum]]-wrapper
- Для больших JSON — `.convertFromSnakeCase` + `.prettyPrinted` только в debug
- Для ошибок — всегда обрабатывай `try` с `catch` (не `try!`)
- Для async — используй [[async]]/[[await]] с [[URLSession]] + `JSONDecoder`
- Для безопасности — проверяй `DecodingError` и логируй `debugDescription`

**Короткое правило**:
> «Decodable — один из самых мощных инструментов [[Swift]].  
> Делай модели `Decodable`, используй `CodingKeys`, кастомные стратегии и `async/await` — и работа с сетью/данными станет почти бесплатной.»
