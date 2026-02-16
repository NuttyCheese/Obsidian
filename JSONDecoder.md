**JSONDecoder** — это класс из Foundation, который позволяет декодировать JSON-данные в Swift-структуры, соответствующие протоколу **Decodable** (или **Codable**).

По состоянию на 2026 год (Swift 6+) `JSONDecoder` остаётся **основным и самым надёжным** инструментом для парсинга JSON в iOS/macOS-приложениях. Он полностью поддерживает современный Swift (async/await, strict concurrency, generics, property wrappers) и активно развивается Apple.

### Основные возможности JSONDecoder (актуальные 2026)

| Возможность / Настройка               | Описание / Значение по умолчанию                          | Рекомендация 2026 года |
|----------------------------------------|------------------------------------------------------------|-------------------------|
| `dateDecodingStrategy`                 | `.deferredToDate` (ISO 8601)                               | `.iso8601` или `.custom` |
| `dataDecodingStrategy`                 | `.deferredToData` (Base64)                                 | `.base64` (чаще всего) |
| `nonConformingFloatDecodingStrategy`   | `.throw` (при NaN, +inf, -inf)                             | `.convertFromString` для API |
| `keyDecodingStrategy`                  | `.useDefaultKeys` (camelCase → snake_case и т.д.)          | `.convertFromSnakeCase` (самый частый) |
| `userInfo`                             | Словарь для передачи контекста (например, DateFormatter)   | Часто используется |
| `decoding` async/await                 | Полная поддержка `JSONDecoder().decode(_:from:)` в async   | Основной способ в 2026 |

### Самые популярные и рекомендуемые паттерны 2026 года

#### Паттерн 1 — Базовый декодинг (самый частый)

```swift
struct User: Codable {
    let id: UUID
    let name: String
    let email: String?
    let createdAt: Date
}

func fetchUser() async throws -> User {
    let url = URL(string: "https://api.example.com/user")!
    let (data, _) = try await URLSession.shared.data(from: url)
    
    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    decoder.keyDecodingStrategy = .convertFromSnakeCase
    
    return try decoder.decode(User.self, from: data)
}
```

#### Паттерн 2 — Кастомный DateFormatter (очень частый для API)

```swift
let decoder = JSONDecoder()

decoder.dateDecodingStrategy = .formatted({
    let formatter = DateFormatter()
    formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
    formatter.locale = Locale(identifier: "en_US_POSIX")
    formatter.timeZone = TimeZone(secondsFromGMT: 0)
    return formatter
}())
```

#### Паттерн 3 — Обработка нестандартных чисел (NaN, Infinity)

```swift
decoder.nonConformingFloatDecodingStrategy = .convertFromString(
    positiveInfinity: "Infinity",
    negativeInfinity: "-Infinity",
    nan: "NaN"
)
```

#### Паттерн 4 — Декодинг с userInfo (для передачи контекста)

```swift
struct User: Decodable {
    let id: UUID
    let name: String
    let role: Role
    
    enum Role: String, Decodable {
        case admin, user, guest
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        id = try container.decode(UUID.self, forKey: .id)
        name = try container.decode(String.self, forKey: .name)
        
        // Используем userInfo
        if let isAdmin = decoder.userInfo[.isAdminKey] as? Bool, isAdmin {
            role = .admin
        } else {
            role = try container.decode(Role.self, forKey: .role)
        }
    }
    
    static let isAdminKey = CodingUserInfoKey(rawValue: "isAdmin")!
}

// Использование
decoder.userInfo[User.isAdminKey] = true
let user = try decoder.decode(User.self, from: data)
```

### Лучшие практики JSONDecoder в Swift 2026

- **Всегда задавай `keyDecodingStrategy = .convertFromSnakeCase`** — 90% современных API используют snake_case  
- **dateDecodingStrategy = .iso8601** — самый надёжный и быстрый для ISO 8601 дат  
- **nonConformingFloatDecodingStrategy** — обязательно для API, которые могут возвращать "Infinity" / "NaN"  
- **userInfo** — используй для передачи контекста (DateFormatter, флаги, environment)  
- **Swift 6 strict concurrency** — `JSONDecoder` thread-safe, но декодируй в `@MainActor` или `Task`  
- **Ошибки** — всегда обрабатывай `DecodingError` с `debugDescription` для логов  
- **Производительность** — для больших JSON используй `JSONDecoder().decode` в `Task.detached`  
- **Тестирование** — создавай JSON-файлы в Bundle и тестируй `decode` на них  
- **Документируйте** — пиши комментарий «JSONDecoder — настроен на snake_case + ISO 8601»

**Короткий девиз 2026**:
> «JSONDecoder в 2026 году — это когда ты хочешь превратить JSON в красивые Swift-структуры быстро, безопасно и без боли.  
> Самый важный момент — правильно настроить `keyDecodingStrategy` и `dateDecodingStrategy`.  
> Для всего остального — `Codable` + `JSONDecoder` — это стандарт.»

Удачи с надёжным и читаемым парсингом JSON в Swift! 📦