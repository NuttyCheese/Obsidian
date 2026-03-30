**NSData** — это класс из **[[Foundation]]**, представляющий **неизменяемую** (immutable) последовательность байтов. Он используется для работы с сырыми данными (бинарными, файлами, сетевыми ответами, криптографией, изображениями и т.д.).

В 2026 году ([[Swift]] 6+) **NSData** всё ещё активно используется, но в чистом Swift-коде его почти полностью вытеснил **Data** — Swift-структура, которая является **[[Value Type]]** и имеет те же возможности, но гораздо лучше интегрируется с современным Swift (generic, [[Codable]], [[async]]/[[await]], strict concurrency).

### Ключевые отличия NSData vs Data (2026 актуальные)

| Характеристика                        | NSData (Objective-C)           | [[Data]] (Swift)                                    | Победитель в 2026                 |
| ------------------------------------- | ------------------------------ | --------------------------------------------------- | --------------------------------- |
| Тип                                   | [[Reference Type]] ([[class]]) | [[Value Type]] ([[struct]])                         | **Data** (копируется по значению) |
| ARC / Memory management               | [[ARC]] (автоматический)       | Автоматический (копирование при необходимости)      | **Data** (без retain cycle)       |
| Mutability                            | Есть `NSMutableData`           | Есть `Data(mutating:)` / `withUnsafeMutableBytes`   | **Data** (более гибко)            |
| Bridging с Swift                      | Полное (toll-free bridged)     | Нативный тип                                        | **Data** (нет overhead)           |
| [[Codable]] / [[JSONEncoder]]/Decoder | Не поддерживает напрямую       | Полная поддержка                                    | **Data**                          |
| [[Swift Concurrency]] ([[Sendable]])  | Не Sendable                    | Sendable                                            | **Data**                          |
| Производительность копирования        | Дешевле (только указатель)     | Дороже при копировании (но CoW — [[Copy-On-Write]]) | Ничья (CoW в Data оптимизирован)  |
| Использование в новом коде            | Только в ObjC-API и legacy     | Почти 100% случаев                                  | **Data**                          |

### Когда NSData всё ещё нужен в 2026 году

| Сценарий                                                                                            | Почему NSData / NSMutableData всё ещё используется      | Рекомендация / альтернатива                             |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| Вызовы **[[Objective-C]] [[API]]** (Core Foundation, Security, Core Audio, [[AVFoundation]] и т.д.) | Многие методы возвращают/принимают NSData/NSMutableData | Приводи сразу к `Data`: `data as Data`                  |
| **NSCoding / NSSecureCoding** (архивация старых объектов)                                           | `encodeObject(_:forKey:)` ожидает NSData                | Переходи на `Codable` + `JSONEncoder`                   |
| **NSKeyedArchiver / NSKeyedUnarchiver**                                                             | Legacy-архивация ([[Swift/Теория/Хранение данных/UserDefaults]] старого стиля)       | Используй `Codable` + `PropertyListEncoder`             |
| **[[Core Data]]** (transformable attributes)                                                        | Иногда хранит NSData                                    | Используй `Codable` атрибуты                            |
| **Высокая совместимость с C/Objective-C**                                                           | Прямой bridging без overhead                            | Редко (почти всегда Data)                               |
| **NSMutableData** для мутабельных буферов                                                           | Динамическое добавление байтов в старом коде            | `Data(mutating:)` + `append` / `withUnsafeMutableBytes` |

### Самые популярные и рекомендуемые паттерны 2026 года

#### Паттерн 1 — Получение NSData из ObjC API и приведение к Data

```swift
let nsData: NSData = someObjCObject.dataValue()

// Самый безопасный и современный способ
let swiftData = nsData as Data

// Или с проверкой
guard let swiftData = nsData as? Data else {
    throw DataError.invalidConversion
}
```

#### Паттерн 2 — Работа с NSMutableData (редко, но встречается)

```swift
let mutableData = NSMutableData()

mutableData.append("Hello".data(using: .utf8)!)
mutableData.append([0x01, 0x02, 0x03] as Data)

// Приведение к Swift Data
let swiftData = mutableData as Data
```

#### Паттерн 3 — Современный стиль: только Data (без NSData)

```swift
func loadImageData(from url: URL) async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Работа с байтами
data.withUnsafeBytes { buffer in
    // низкоуровневая обработка
}
```

### Лучшие практики работы с NSData в Swift 2026

- **Избегай NSData в чистом Swift-коде** — используй `Data`  
- **При получении NSData из ObjC** — сразу приводи к `Data` (`as Data` или `as? Data`)  
- **NSMutableData** — заменяй на `Data(mutating:)` + `append` / `withUnsafeMutableBytes`  
- **Swift 6 strict concurrency** — `NSData` **не Sendable**, поэтому передавай через `@MainActor` или копируй в `Data`  
- **Тестирование** — моки NSData через `Data(base64Encoded:)` или `Data([UInt8])`  
- **Документируйте** — пиши комментарий «NSData из Objective-C API — приводим к Data»

**Короткий девиз 2026**:
> «NSData в 2026 году — это когда ты вынужден работать с Objective-C миром или legacy-кодом.  
> В чистом Swift почти всё заменено на Data — value type, Sendable, Codable и гораздо удобнее.  
> Главное правило: как только получил NSData — сразу приводи к Data и забудь про него.»
