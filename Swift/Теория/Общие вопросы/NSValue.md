**NSValue** — это класс из **Foundation**, который служит **обёрткой** (wrapper) для упаковки **примитивных типов** (C-структур, чисел, указателей и т.д.) в объект [[Objective-C]]. Он позволяет хранить не-объектные значения в коллекциях Objective-C ([[NSArray]], [[NSDictionary]], [[NSSet]] и т.д.), где требуются объекты.

В 2026 году (Swift 6+, iOS 18+, macOS 15+) **NSValue** всё ещё активно используется, но в чистом [[Swift]]-коде его роль **значительно сократилась** в пользу **нативных Swift-типов** и **Swift-структур**.  
NSValue встречается почти исключительно в ситуациях, связанных с **Objective-C совместимостью** и legacy-[[API]].

### Ключевые отличия NSValue vs Swift-альтернативы (2026 актуальные)

| Характеристика                        | NSValue (Objective-C)                  | Swift-альтернатива (2026)                      | Победитель в 2026 |
| ------------------------------------- | -------------------------------------- | ---------------------------------------------- | ----------------- |
| Тип                                   | [[Reference type]] ([[class]])         | [[Value type]] ([[struct]] / [[enum]])         | **Swift**         |
| [[ARC]] / Memory management           | [[ARC]]                                | Автоматический (копирование при необходимости) | **Swift**         |
| Bridging с Swift                      | Полное (toll-free bridged)             | Нативные типы                                  | **Swift**         |
| [[Codable]] / [[JSONEncoder]]/Decoder | Не поддерживает напрямую               | Полная поддержка                               | **Swift**         |
| [[Swift Concurrency]] (Sendable)      | Не [[Sendable]]                        | Sendable (если тип Sendable)                   | **Swift**         |
| Использование в новом коде            | Только в ObjC-API и legacy             | Почти 100% случаев                             | **Swift**         |
| Основное назначение                   | Упаковка примитивов для ObjC-коллекций | Прямое использование типов                     | **Swift**         |

### Когда NSValue всё ещё нужен в 2026 году (реальные сценарии)

| Сценарий                                                                                                                                       | Почему NSValue используется                                                                       | Рекомендация / альтернатива                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Вызовы **Objective-C API** ([[UIKit]], [[AppKit]], Core [[Foundation]], Core Animation, Core Graphics, [[Core Data]], [[UserDefaults]] и т.д.) | Многие методы возвращают/принимают NSValue (например, [[CGPoint]], [[CGRect]], CGAffineTransform) | Приводи сразу к Swift-типу: `value.cgPointValue`, `value.cgRectValue` |
| **UserDefaults** (стандартный способ хранения структур)                                                                                        | `object(forKey:)` может вернуть NSValue для CGPoint, CGRect и т.д.                                | Переходи на `Codable` + JSON в UserDefaults                           |
| **Core Animation** (CAAnimation, CALayer)                                                                                                      | `setValue(_:forKey:)` часто принимает NSValue для CGPoint, CGRect, CATransform3D                  | Используй `CGPoint`, `CGRect` напрямую                                |
| **Core Graphics** (CGGeometry)                                                                                                                 | `NSValue` оборачивает CGPoint, CGSize, CGRect, CGAffineTransform                                  | Используй нативные Swift-структуры                                    |
| **[[KVC]] / [[KVO]]** (value(forKey:))                                                                                                         | Возвращает NSValue для структурных свойств                                                        | Combine / Observation (Swift 5.9+)                                    |
| **Legacy-код** / поддержка iOS 12–14                                                                                                           | Совместимость                                                                                     | Миграция на Swift-структуры                                           |
| **Высокая совместимость с C/Objective-C**                                                                                                      | Прямой bridging без overhead                                                                      | Редко (почти всегда Swift-типы)                                       |

### Самые популярные и рекомендуемые паттерны NSValue → Swift в 2026

#### Паттерн 1 — Получение NSValue из ObjC API и распаковка в Swift-структуру

```swift
let nsValue: NSValue = someObjCObject.value(forKey: "frame")

// Самый безопасный и современный способ
let cgRect = nsValue.cgRectValue
let cgPoint = nsValue.cgPointValue
let cgSize = nsValue.cgSizeValue
let transform = nsValue.caTransform3DValue

// Или с проверкой
guard let rect = nsValue.cgRectValue else {
    throw ValueError.invalidRect
}
```

#### Паттерн 2 — Создание NSValue для передачи в ObjC API

```swift
let rect = CGRect(x: 0, y: 0, width: 100, height: 100)
let nsRect = NSValue(cgRect: rect)

// Передача в ObjC-метод
layer.setValue(nsRect, forKey: "frame")
```

#### Паттерн 3 — Современный стиль: только Swift-структуры (без NSValue)

```swift
struct FrameInfo {
    let rect: CGRect
    let center: CGPoint
}

func processFrame(_ frame: CGRect) {
    let center = CGPoint(x: frame.midX, y: frame.midY)
    // работаем напрямую с CGRect и CGPoint
}
```

### Лучшие практики NSValue в Swift 2026

- **Избегай NSValue в чистом Swift-коде** — используй `CGPoint`, `CGSize`, `CGRect`, `CGAffineTransform`, `CATransform3D` и т.д.  
- **При получении NSValue из ObjC** — сразу распаковывай через `.cgRectValue`, `.cgPointValue` и т.д.  
- **Swift 6 strict concurrency** — `NSValue` **не Sendable**, поэтому передавай через `@MainActor` или копируй в Swift-структуры  
- **Тестирование** — моки NSValue через `NSValue(cgRect: .zero)`  
- **Документируйте** — пиши комментарий «NSValue из Objective-C API — распаковываем в CGRect»

**Короткий девиз 2026**:
> «NSValue в 2026 году — это когда Objective-C API хочет передать структуру как объект.  
> В чистом Swift почти всё заменено на CGPoint, CGRect, CGSize и т.д. — value type, Sendable и гораздо удобнее.  
> Главное правило: как только получил NSValue — сразу распаковывай в Swift-структуру и забудь про него.»
