**NSString** — это основной класс в **Foundation** для работы со строками в Objective-C и Swift (до появления нативного `String`).

В 2026 году (Swift 6+, iOS 18+, macOS 15+) **NSString** всё ещё очень активно используется, но в чистом Swift-коде его роль **сильно уменьшилась** в пользу **String** — нативного Swift-типа, который:

- является **value type** (копируется по значению, без retain cycle)  
- полностью **Sendable** (безопасен в Swift Concurrency)  
- лучше интегрируется с generics, Codable, SwiftUI, Combine, async/await  
- имеет более удобный, выразительный и безопасный API  
- поддерживает Unicode нативно (UTF-8 по умолчанию)

### Ключевые отличия NSString vs String (актуальные в 2026)

| Характеристика                  | NSString (Objective-C)                         | String (Swift)                                   | Победитель в 2026 |
|---------------------------------|------------------------------------------------|--------------------------------------------------|-------------------|
| Тип                             | Reference type (class)                         | Value type (struct)                              | **String**        |
| Управление памятью              | ARC                                            | Автоматическое (CoW — copy-on-write)             | **String**        |
| Мутабельность                   | Есть `NSMutableString`                         | Mutable по умолчанию (var str)                   | **String**        |
| Bridging с Swift                | Полное (toll-free bridged)                     | Нативный тип                                     | **String**        |
| Codable / JSONEncoder/Decoder   | Поддерживает через bridging                    | Полная поддержка                                 | **String**        |
| Swift Concurrency (Sendable)    | Не Sendable                                    | Sendable                                         | **String**        |
| Производительность копирования  | Дешевле (указатель)                            | Дешевле при использовании (CoW)                  | Ничья             |
| Unicode и UTF-8                 | UTF-16 (UTF-16 кодовые единицы)                | UTF-8 (нативно)                                  | **String**        |
| Использование в новом коде      | Только в ObjC-API и legacy                     | Почти 100% случаев                               | **String**        |

### Когда NSString всё ещё нужен в 2026 году (реальные сценарии)

| Сценарий                                      | Почему NSString всё ещё используется | Рекомендация / альтернатива |
|-----------------------------------------------|---------------------------------------|-----------------------------|
| Вызовы **Objective-C API** (UIKit, AppKit, Core Foundation, Core Text, Security, NSUserDefaults, KVC/KVO и т.д.) | Многие методы возвращают/принимают NSString/NSMutableString | Приводи сразу к `String`: `string as String` |
| **UserDefaults** (стандартный способ хранения строк) | `string(forKey:)` возвращает String?, но иногда приходит NSString | Используй `string(forKey:)` напрямую |
| **KVC / KVO** (value(forKey:))                | Часто возвращает NSString                      | Combine / Observation (Swift 5.9+) |
| **Core Foundation** (CFStringRef)             | CFStringRef toll-free bridged с NSString       | Приводи к `String` |
| **NSStringDrawing** / **NSAttributedString**  | Методы рисования и атрибутов работают с NSString | `NSAttributedString` + `String` |
| **Legacy-код** / поддержка iOS 12–14          | Совместимость                                  | Миграция на String          |
| **Высокая совместимость с C/Objective-C**     | Прямой bridging без overhead                   | Редко (почти всегда String) |

### Самые популярные и рекомендуемые паттерны NSString → String в 2026

#### Паттерн 1 — Получение NSString из ObjC API и приведение к String

```swift
let nsString: NSString = someObjCObject.stringValue()

// Самый безопасный и современный способ
let swiftString = nsString as String

// Или с проверкой
guard let swiftString = nsString as? String else {
    throw StringError.invalidConversion
}
```

#### Паттерн 2 — Работа с NSMutableString (редко, но встречается)

```swift
let mutableString = NSMutableString(string: "Hello")

mutableString.append(" World")
mutableString.insert("!", at: 5)

// Приведение к Swift String
let swiftString: String = mutableString as String
```

#### Паттерн 3 — Современный стиль: только String (без NSString)

```swift
func formatGreeting(name: String) -> String {
    "Hello, \(name)!"  // чистый Swift String
}

let greeting = formatGreeting(name: "Alex")
print(greeting)
```

### Лучшие практики NSString в Swift 2026

- **Избегай NSString в чистом Swift-коде** — используй `String`  
- **При получении NSString из ObjC** — сразу приводи к `String` (`as String` или `as? String`)  
- **NSMutableString** → заменяй на `var str = ""` + `+=`, `append`, `insert`  
- **Swift 6 strict concurrency** — `NSString` **не Sendable**, поэтому передавай через `@MainActor` или копируй в `String`  
- **Тестирование** — моки NSString через `NSString(string: "test")`  
- **Документируйте** — пиши комментарий «NSString из Objective-C API — приводим к String»

**Короткий девиз 2026**:
> «NSString в 2026 году — это когда Objective-C API хочет передать строку как объект.  
> В чистом Swift почти всё заменено на String — value type, Sendable, Codable и гораздо удобнее.  
> Главное правило: как только получил NSString — сразу приводи к String и забудь про него.»

Удачи с типобезопасными и современными строками в Swift! 📜