**NSDictionary** — это **неизменяемый** (immutable) словарь (ассоциативный массив) из **[[Foundation]]**, который используется для хранения пар «ключ-значение» в [[Objective-C]] и [[Swift]].

В 2026 году (Swift 6+, iOS 18+, macOS 15+) **NSDictionary** всё ещё активно применяется, но в чистом Swift-коде его использование **сильно сократилось** в пользу **Swift Dictionary** (`[Key: Value]`), потому что последний:

- **типобезопасен** (Key: [[Hashable]], Value: любой тип)  
- является **[[value type]]** (копируется по значению, без [[retain cycle]])  
- полностью **[[Sendable]]** (безопасен в [[Swift Concurrency]])  
- лучше интегрируется с [[generic]], [[Codable]], [[SwiftUI]], [[Combine]], [[async]]/[[await]]  
- имеет более удобный и выразительный API

### Ключевые отличия NSDictionary vs Dictionary (2026 актуальные)

| Характеристика                        | NSDictionary (Objective-C)     | Dictionary<Key: Hashable, Value> (Swift)      | Победитель в 2026 |
| ------------------------------------- | ------------------------------ | --------------------------------------------- | ----------------- |
| Тип                                   | [[Reference type]] ([[class]]) | [[Value type]] ([[struct]])                   | **Dictionary**    |
| ARC / Memory management               | [[ARC]]                        | Автоматический (CoW — [[copy-on-write]])      | **Dictionary**    |
| Mutability                            | Есть `NSMutableDictionary`     | Mutable по умолчанию (var dict)               | **Dictionary**    |
| Bridging с Swift                      | Полное (toll-free bridged)     | Нативный тип                                  | **Dictionary**    |
| [[Codable]] / [[JSONEncoder]]/Decoder | Не поддерживает напрямую       | Полная поддержка                              | **Dictionary**    |
| Swift Concurrency (Sendable)          | Не Sendable                    | Sendable (если Key и Value Sendable)          | **Dictionary**    |
| Производительность копирования        | Дешевле (только указатель)     | Дороже при копировании (но CoW оптимизирован) | Ничья             |
| Использование в новом коде            | Только в ObjC-API и legacy     | Почти 100% случаев                            | **Dictionary**    |

### Когда NSDictionary всё ещё нужен в 2026 году (реальные сценарии)

| Сценарий                                                                                                                                 | Почему NSDictionary / NSMutableDictionary всё ещё используется      | Рекомендация / альтернатива                              |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------- |
| Вызовы **[[Objective-C]] [[API]]** ([[UIKit]], [[AppKit]], Core [[Foundation]], [[UserDefaults]], [[Core Data]], [[KVC]]/[[KVO]] и т.д.) | Многие методы возвращают/принимают NSDictionary/NSMutableDictionary | Приводи сразу к `[AnyHashable: Any]` или `[String: Any]` |
| **UserDefaults** (стандартный способ хранения словарей)                                                                                  | `dictionary(forKey:)` возвращает NSDictionary?                      | Переходи на `Codable` + JSON в UserDefaults              |
| **KVC / KVO** (Key-Value Coding / Observing)                                                                                             | `value(forKeyPath:)` часто возвращает NSDictionary                  | Combine / Observation (Swift 5.9+)                       |
| **NSCoding / NSSecureCoding** (архивация старых объектов)                                                                                | `encodeObject(_:forKey:)` ожидает NSDictionary                      | Переходи на `Codable` + `JSONEncoder`                    |
| **Core Data** (transformable attributes или relationships)                                                                               | Иногда хранит NSDictionary                                          | Используй `Codable` атрибуты                             |
| **Legacy-код** / поддержка iOS 12–14                                                                                                     | Совместимость                                                       | Миграция на Dictionary                                   |
| **Высокая совместимость с C/Objective-C**                                                                                                | Прямой bridging без overhead                                        | Редко (почти всегда Dictionary)                          |

### Самые популярные и рекомендуемые паттерны NSDictionary → Dictionary в 2026

#### Паттерн 1 — Получение NSDictionary из ObjC API и приведение к Swift Dictionary

```swift
let nsDict: NSDictionary = someObjCObject.dictionaryValue()

// Самый безопасный и современный способ
let swiftDict = nsDict as? [String: Any] 
    ?? nsDict.reduce(into: [String: Any]()) { result, pair in
        if let key = pair.key as? String {
            result[key] = pair.value
        }
    }

// Или с конкретным типом значений
let stringDict = nsDict as? [String: String]
```

#### Паттерн 2 — Работа с NSMutableDictionary (редко, но встречается)

```swift
let mutableDict = NSMutableDictionary()

mutableDict["name"] = "Alice"
mutableDict["age"] = 30
mutableDict.setValue("active", forKey: "status")

// Приведение к Swift Dictionary
let swiftDict: [String: Any] = mutableDict as [String: Any]
```

#### Паттерн 3 — Современный стиль: только [[Dictionary]] (без NSDictionary)

```swift
func parseUserInfo(data: Data) throws -> [String: Any] {
    guard let dict = try JSONSerialization.jsonObject(with: data) as? [String: Any] else {
        throw DecodingError.invalidJSON
    }
    return dict
}

// Или с Codable (предпочтительно)
struct UserInfo: Codable {
    let name: String
    let age: Int
    let status: String?
}
```

### Лучшие практики NSDictionary в Swift 2026

- **Избегай NSDictionary в чистом Swift-коде** — используй `[Key: Value]`  
- **При получении NSDictionary из ObjC** — сразу приводи к `[String: Any]` или `[AnyHashable: Any]`  
- **compactMap / reduce** — для безопасного приведения типов  
- **Swift 6 strict concurrency** — `NSDictionary` **не Sendable**, поэтому передавай через `@MainActor` или копируй в `[String: Any]`  
- **Тестирование** — моки NSDictionary через `NSDictionary(dictionary: [...])`  
- **Документируйте** — пиши комментарий «NSDictionary из Objective-C API — приводим к [String: Any]»

**Короткий девиз 2026**:
> «NSDictionary в 2026 году — это когда ты вынужден работать с Objective-C миром или legacy-кодом.  
> В чистом Swift почти всё заменено на Dictionary<Key, Value> — value type, Sendable, Codable и гораздо удобнее.  
> Главное правило: как только получил NSDictionary — сразу приводи к [String: Any] и забудь про него.»
