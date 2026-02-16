**NSNumber** — это класс из **Foundation**, который представляет собой **обёртку** (wrapper) над примитивными числовыми типами (Int, Float, Double, Bool и т.д.) в Objective-C.

В 2026 году (Swift 6+, iOS 18+, macOS 15+) **NSNumber** всё ещё активно используется, но в чистом Swift-коде его применение **сильно сократилось** в пользу **нативных Swift-типов** (`Int`, `Double`, `Float`, `Bool`, `Decimal` и т.д.), потому что они:

- являются **value type** (копируются по значению, без retain cycle)  
- полностью **Sendable** (безопасны в Swift Concurrency)  
- лучше интегрируются с generics, Codable, SwiftUI, Combine, async/await  
- имеют более удобный и выразительный API

### Ключевые отличия NSNumber vs Swift-примитивы (2026 актуальные)

| Характеристика                  | NSNumber (Objective-C)                         | Swift-примитивы (Int, Double, Bool и т.д.) | Победитель в 2026 |
|---------------------------------|------------------------------------------------|---------------------------------------------|-------------------|
| Тип                             | Reference type (class)                         | Value type                                  | **Swift-примитивы** |
| ARC / Memory management         | ARC                                            | Автоматический (копирование при необходимости) | **Swift-примитивы** |
| Mutability                      | Mutable (NSNumber всегда immutable, но есть NSMutableNumber в редких случаях) | Immutable по умолчанию (var для мутации)    | **Swift-примитивы** |
| Bridging с Swift                | Полное (toll-free bridged)                     | Нативные типы                               | **Swift-примитивы** |
| Codable / JSONEncoder/Decoder   | Поддерживает через NSNumber bridging           | Полная поддержка                            | **Swift-примитивы** |
| Swift Concurrency (Sendable)    | Не Sendable                                    | Sendable                                    | **Swift-примитивы** |
| Производительность              | Дешевле при копировании (указатель)            | Дешевле при использовании (нет боксинга)    | **Swift-примитивы** |
| Использование в новом коде      | Только в ObjC-API и legacy                     | Почти 100% случаев                          | **Swift-примитивы** |

### Когда NSNumber всё ещё нужен в 2026 году (реальные сценарии)

| Сценарий                                      | Почему NSNumber всё ещё используется | Рекомендация / альтернатива |
|-----------------------------------------------|---------------------------------------|-----------------------------|
| Вызовы **Objective-C API** (UIKit, AppKit, Core Foundation, UserDefaults, Core Data, KVC/KVO, NSNumberFormatter и т.д.) | Многие методы возвращают/принимают NSNumber | Приводи сразу к Swift-типу: `number as? Int` / `number.doubleValue` |
| **UserDefaults** (стандартный способ хранения чисел) | `object(forKey:)` может вернуть NSNumber | Используй `integer(forKey:)`, `double(forKey:)` или `Codable` |
| **KVC / KVO** (value(forKey:))                | Часто возвращает NSNumber для числовых свойств | Combine / Observation (Swift 5.9+) |
| **Core Data** (numeric attributes)            | Числовые атрибуты хранятся как NSNumber       | Используй `@NSManaged` с Swift-типами или `Codable` |
| **NSNumberFormatter** / форматирование чисел  | Работает только с NSNumber                    | `NumberFormatter` + Swift-типы (Double, Int) |
| **Legacy-код** / поддержка iOS 12–14          | Совместимость                                 | Миграция на Swift-примитивы |
| **Высокая совместимость с C/Objective-C**     | Прямой bridging без overhead                  | Редко (почти всегда Swift-типы) |

### Самые популярные и рекомендуемые паттерны NSNumber → Swift в 2026

#### Паттерн 1 — Безопасное приведение NSNumber к Swift-примитивам

```swift
let nsNumber: NSNumber = someObjCObject.numberValue()

// Самый безопасный и современный способ
let intValue = nsNumber.intValue          // или .int64Value, .doubleValue
let doubleValue = nsNumber.doubleValue

// Или с проверкой типа
if let intValue = nsNumber as? Int {
    // ...
} else if let doubleValue = nsNumber as? Double {
    // ...
}
```

#### Паттерн 2 — Работа с UserDefaults и NSNumber (legacy, но всё ещё живой)

```swift
let defaults = UserDefaults.standard

// Запись
defaults.set(42 as NSNumber, forKey: "answer")
defaults.set(3.14 as NSNumber, forKey: "pi")

// Чтение
let answer = defaults.integer(forKey: "answer")     // 42
let pi = defaults.double(forKey: "pi")              // 3.14

// Или через object(forKey:)
if let value = defaults.object(forKey: "answer") as? NSNumber {
    print(value.intValue)
}
```

**Лучшая практика 2026** — не храни числа в UserDefaults как NSNumber.  
Используй **Codable** + **JSONEncoder/Decoder** или прямые методы (`integer(forKey:)`, `double(forKey:)`).

#### Паттерн 3 — Современный стиль: только Swift-примитивы (без NSNumber)

```swift
func processScore(_ score: Int) {
    // Никакого NSNumber
    print("Score: \(score)")
}

let scoreFromAPI: Int = try decoder.decode(Int.self, from: data)
// или
let score = response["score"] as? Int ?? 0
```

### Лучшие практики NSNumber в Swift 2026

- **Избегай NSNumber в чистом Swift-коде** — используй `Int`, `Double`, `Float`, `Bool`, `Decimal`  
- **При получении NSNumber из ObjC** — сразу приводи к Swift-типу (`as? Int`, `.intValue`, `.doubleValue`)  
- **UserDefaults** — используй `integer(forKey:)`, `double(forKey:)` вместо `object(forKey:)`  
- **Swift 6 strict concurrency** — `NSNumber` **не Sendable**, поэтому передавай через `@MainActor` или копируй в Swift-примитивы  
- **Тестирование** — моки NSNumber через `NSNumber(value: 42)`  
- **Документируйте** — пиши комментарий «NSNumber из Objective-C API — приводим к Int/Double»

**Короткий девиз 2026**:
> «NSNumber в 2026 году — это когда Objective-C API хочет передать число как объект.  
> В чистом Swift почти всё заменено на Int, Double, Bool и т.д. — value type, Sendable, Codable и гораздо удобнее.  
> Главное правило: как только получил NSNumber — сразу приводи к Swift-примитиву и забудь про него.»

Удачи с типобезопасными и современными числами в Swift! 🔢