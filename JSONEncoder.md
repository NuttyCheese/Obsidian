**JSONEncoder** — это класс из Foundation, который позволяет кодировать (преобразовывать) Swift-объекты, соответствующие протоколу **Encodable** (или **Codable**), в JSON-данные.

По состоянию на 2026 год (Swift 6+) `JSONEncoder` остаётся **основным и самым надёжным** инструментом для сериализации данных в JSON в iOS/macOS-приложениях. Он полностью поддерживает современный Swift (async/await, strict concurrency, property wrappers, generics) и активно используется Apple.

### Основные возможности JSONEncoder (актуальные 2026)

| Свойство / Стратегия                     | Значение по умолчанию                              | Рекомендация 2026 года                          | Когда менять |
|------------------------------------------|-----------------------------------------------------|--------------------------------------------------|--------------|
| `outputFormatting`                       | `[]` (компактный JSON)                              | `.prettyPrinted` для отладки, `.withoutEscapingSlashes` для чистоты | Отладка / API |
| `dateEncodingStrategy`                   | `.deferredToDate` (ISO 8601)                        | `.iso8601` или `.formatted` с кастомным DateFormatter | Почти всегда |
| `dataEncodingStrategy`                   | `.deferredToData` (Base64)                          | `.base64` (стандарт для байтов)                  | Почти всегда |
| `nonConformingFloatEncodingStrategy`     | `.throw` (при NaN, +inf, -inf)                      | `.convertToString` для совместимости с API       | При необходимости |
| `keyEncodingStrategy`                    | `.useDefaultKeys`                                   | `.convertToSnakeCase` (самый частый для API)     | Почти всегда |
| `userInfo`                               | Пустой словарь                                      | Для передачи контекста (DateFormatter и т.д.)    | Часто |
| `sortedKeys` (iOS 16+)                   | `false`                                             | `true` для стабильного порядка ключей            | Отладка / кэширование |

### Самые популярные и рекомендуемые паттерны 2026 года

#### Паттерн 1 — Базовое кодирование (самый частый)

```swift
struct User: Codable {
    let id: UUID
    let name: String
    let email: String?
    let createdAt: Date
}

func encodeUser(_ user: User) throws -> Data {
    let encoder = JSONEncoder()
    encoder.dateEncodingStrategy = .iso8601
    encoder.keyEncodingStrategy = .convertToSnakeCase
    encoder.outputFormatting = [.prettyPrinted, .sortedKeys] // для отладки
    
    return try encoder.encode(user)
}
```

#### Паттерн 2 — Кастомный DateFormatter (очень частый для API)

```swift
let encoder = JSONEncoder()

encoder.dateEncodingStrategy = .formatted({
    let formatter = ISO8601DateFormatter()
    formatter.formatOptions = [.withInternetDateTime, .withFractionalSeconds]
    formatter.timeZone = TimeZone(secondsFromGMT: 0)
    return formatter
}())
```

#### Паттерн 3 — Кодирование с userInfo (для передачи контекста)

```swift
struct User: Encodable {
    let id: UUID
    let name: String
    let role: Role
    
    enum Role: String, Encodable {
        case admin, user, guest
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(id, forKey: .id)
        try container.encode(name, forKey: .name)
        
        // Используем userInfo
        if let isAdmin = encoder.userInfo[.isAdminKey] as? Bool, isAdmin {
            try container.encode("admin", forKey: .role)
        } else {
            try container.encode(role.rawValue, forKey: .role)
        }
    }
    
    static let isAdminKey = CodingUserInfoKey(rawValue: "isAdmin")!
}

// Использование
encoder.userInfo[User.isAdminKey] = true
let data = try encoder.encode(user)
```

#### Паттерн 4 — Асинхронное кодирование в Task (очень частый в 2026)

```swift
func sendUserToServer(_ user: User) async throws {
    let encoder = JSONEncoder()
    encoder.keyEncodingStrategy = .convertToSnakeCase
    
    let data = try encoder.encode(user)
    
    var request = URLRequest(url: URL(string: "https://api.example.com/users")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = data
    
    let (responseData, _) = try await URLSession.shared.data(for: request)
    // обработка ответа
}
```

### Лучшие практики JSONEncoder в Swift 2026

- **keyEncodingStrategy = .convertToSnakeCase** — 90% современных API используют snake_case  
- **dateEncodingStrategy = .iso8601** — самый надёжный и быстрый для большинства API  
- **outputFormatting = [.prettyPrinted, .sortedKeys]** — только для отладки / логов (убирай в продакшене)  
- **nonConformingFloatEncodingStrategy** — обязательно для API, которые могут принимать "Infinity" / "NaN"  
- **userInfo** — используй для передачи контекста (DateFormatter, флаги, environment)  
- **Swift 6 strict concurrency** — `JSONEncoder` thread-safe, но кодируй в `@MainActor` или `Task`  
- **Ошибки** — всегда обрабатывай `EncodingError` с `debugDescription` для логов  
- **Производительность** — для больших объектов используй `encode` в `Task.detached`  
- **Тестирование** — создавай ожидаемый JSON в Bundle и сравнивай `encode` с ним  
- **Документируйте** — пиши комментарий «JSONEncoder — настроен на snake_case + ISO 8601»

**Короткий девиз 2026**:
> «JSONEncoder в 2026 году — это когда ты хочешь превратить красивые Swift-структуры в правильный JSON быстро и без боли.  
> Самый важный момент — правильно настроить `keyEncodingStrategy` и `dateEncodingStrategy`.  
> Для всего остального — `Codable` + `JSONEncoder` — это стандарт.»

Удачи с надёжным и читаемым кодированием JSON в Swift! 📤