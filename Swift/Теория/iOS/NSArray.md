**NSArray** — это **неизменяемый** (immutable) массив объектов в **[[Foundation]]** ([[Objective-C]] и [[Swift]]), который используется для хранения упорядоченной коллекции элементов.

В 2026 году (Swift 6+, [[iOS]] 18+, macOS 15+) **NSArray** остаётся актуальным, но в чистом Swift-коде его использование **сильно сократилось**.  
Большинство разработчиков предпочитают **Swift [[Array]]** (`[T]`), потому что он:

- типобезопасен  
- быстрее в большинстве случаев  
- лучше интегрируется с [[generic]], [[Codable]], [[SwiftUI]], [[Combine]] и [[Swift Concurrency]]  
- имеет более удобный синтаксис

### Когда NSArray всё ещё используется в 2026 году (реальные сценарии)

| Сценарий                                                                             | Почему NSArray / NSMutableArray всё ещё нужен             | Альтернатива в чистом Swift        |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------- | ---------------------------------- |
| Работа с **Objective-C API** ([[UIKit]], AppKit, Core Foundation, старые фреймворки) | Многие методы возвращают/принимают NSArray/NSMutableArray | `as [Any]` / `as? [T]`             |
| **[[KVC]] / [[KVO]]** (Key-Value Coding / Observing)                                 | `value(forKeyPath:)` часто возвращает NSArray             | Combine / Observation (Swift 5.9+) |
| **[[Swift/UserDefaults]]** (стандартный способ хранения массивов)                          | `array(forKey:)` возвращает NSArray?                      | `Codable` + JSON в UserDefaults    |
| **NSCoding / NSSecureCoding** (архивация)                                            | `encodeObject(_:forKey:)` ожидает NSArray                 | `Codable` + JSON / PropertyList    |
| **[[Core Data]]** (managed objects relationships)                                    | To-many relationships — это NSArray                       | `@FetchRequest` / [[SwiftData]]    |
| **Legacy-код** / поддержка iOS 12–14                                                 | Совместимость                                             | Миграция на Swift Array            |
| **Высокая совместимость с C/[[Objective-C]]**                                        | Прямой bridging без overhead                              | Редко                              |

### Самые популярные и рекомендуемые паттерны NSArray в Swift 2026

#### Паттерн 1 — Получение NSArray из Objective-C API и приведение к Swift Array

```swift
let arrayFromObjC: NSArray = someObjCObject.arrayValue()

// Самый безопасный и современный способ
let swiftArray = arrayFromObjC as? [String] 
    ?? arrayFromObjC.compactMap { $0 as? String }

// Или с generics (если уверен в типе)
let users = arrayFromObjC as? [User]

// Самый надёжный вариант (с проверкой)
let safeUsers: [User] = arrayFromObjC.compactMap { $0 as? User }
```

#### Паттерн 2 — Работа с NSMutableArray (редко, но встречается)

```swift
let mutableArray = NSMutableArray()

mutableArray.add("Item 1")
mutableArray.add(42)
mutableArray.add(User(name: "Alice"))

// Приведение к Swift
let mixed: [Any] = mutableArray as [Any]

// Фильтрация только строк
let strings = mutableArray.compactMap { $0 as? String }
```

#### Паттерн 3 — UserDefaults и NSArray (legacy, но всё ещё живой)

```swift
let defaults = UserDefaults.standard

// Запись
defaults.set(["apple", "banana", "cherry"] as NSArray, forKey: "fruits")

// Чтение
if let fruits = defaults.array(forKey: "fruits") as? [String] {
    print(fruits)
}
```

**Лучшая практика 2026** — не храни массивы в UserDefaults как NSArray.  
Используй **Codable** + **JSONEncoder/Decoder**:

```swift
struct Fruits: Codable {
    let items: [String]
}

let fruits = Fruits(items: ["apple", "banana"])
let data = try JSONEncoder().encode(fruits)
UserDefaults.standard.set(data, forKey: "fruits")

// Чтение
if let data = UserDefaults.standard.data(forKey: "fruits"),
   let fruits = try? JSONDecoder().decode(Fruits.self, from: data) {
    print(fruits.items)
}
```

### Лучшие практики работы с NSArray в Swift 2026

- **Избегай NSArray в чистом Swift-коде** — используй `[T]`  
- **При получении NSArray из ObjC** — сразу приводи к `[T]` или `[Any]`  
- **compactMap / map** — для безопасного приведения типов  
- **as? [T]** — самый быстрый и безопасный способ  
- **Swift 6 strict concurrency** — NSArray **не Sendable**, поэтому передавай через `@MainActor` или копируй в `[T]`  
- **Тестирование** — моки NSArray через `NSArray(array: [...])`  
- **Документируйте** — пиши комментарий «NSArray из Objective-C API — приводим к [String]»

**Короткий девиз 2026**:
> «NSArray в 2026 году — это когда ты вынужден работать с Objective-C миром или legacy-кодом.  
> В чистом Swift почти всё заменено на Array<T>.  
> Главное правило: как только получил NSArray — сразу приводи к [T] и забудь про него.»
