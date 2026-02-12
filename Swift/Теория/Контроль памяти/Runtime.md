#memory_control #Swift 
**Swift Runtime** — это низкоуровневая часть стандартной библиотеки [[Swift]], которая отвечает за **динамическое поведение языка** во время выполнения программы.

Она обеспечивает:
- динамическую типизацию и рефлексию
- вызов методов (диспетчеризацию)
- обработку ошибок
- создание и уничтожение объектов
- поддержку протоколов и generics
- безопасное приведение типов (`as?`, `as!`)
- работу с [[Any]], [[AnyObject]], [[AnyClass]]

Swift Runtime — это **не** то же самое, что [[Objective-C]] [[Runtime]].  
С 2019–2020 годов (Swift 5.1+) большая часть динамики переехала на **нативный Swift Runtime**, хотя ObjC-рантайм всё ещё используется для совместимости с Cocoa/[[Foundation]]/[[UIKit]].

### Основные компоненты Swift Runtime (2026)

| Компонент                   | Что делает                                                                 | Когда активно используется                          |
| --------------------------- | -------------------------------------------------------------------------- | --------------------------------------------------- |
| **Type Metadata**           | Хранит информацию о типе: имя, размер, alignment, vtable, протоколы и т.д. | `type(of:)`, `Mirror`, [[generic]], `Any`           |
| **Value Witness Table**     | Функции копирования, перемещения, уничтожения value types                  | [[struct]], [[enum]], [[Array]] (COW), [[String]]   |
| **Protocol Witness Table**  | Таблица реализаций протокола для конкретного типа                          | [[protocol]], [[any]], [[some]], динамический вызов |
| **VTable / Dispatch Table** | Таблица виртуальных методов для классов                                    | [[class]] без `final`, `override`                   |
| **Existential Container**   | Хранит значение `any Protocol` (value buffer + metadata + witness table)   | `any Protocol`, `Any`, `AnyObject`                  |
| **Error Handling**          | Поддержка `throw`, `try`, `catch`, `throws`, `rethrows`                    | Все `throws`-функции                                |
| **Dynamic Casting**         | Реализация `as?`, `as!`, `is`                                              | Приведение типов во время выполнения                |
| **Reflection API**          | [[Mirror]], `CustomReflectable`, `ReflectionMirror`                     | Отладка, сериализация, инспекторы                   |

### Примеры использования Swift Runtime на практике

#### 1. Простая рефлексия (Mirror)

```swift
struct User {
    let name: String
    var age: Int
    private var password: String = "secret"
}

let user = User(name: "Анна", age: 28)

let mirror = Mirror(reflecting: user)
print("Тип:", mirror.subjectType)           // User
print("Количество детей:", mirror.children.count) // 2 (name и age)

for child in mirror.children {
    print("Свойство:", child.label ?? "—", "=", child.value)
}
// name = Анна
// age = 28
// password НЕ видно (private)
```

#### 2. Динамическое создание типа (через Type Metadata)

```swift
func createInstance<T>(of type: T.Type) -> T? where T: ExpressibleByStringLiteral {
    type.init("Hello from runtime")
}

let str = createInstance(of: String.self)   // "Hello from runtime"
```

#### 3. Проверка соответствия протоколу в runtime

```swift
protocol Loggable {
    func log()
}

extension String: Loggable {
    func log() { print("String:", self) }
}

let value: Any = "Test log"
if let loggable = value as? Loggable {
    loggable.log()  // → String: Test log
}
```

#### 4. Witness Table в действии (протоколы)

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

let shape: any Drawable = Circle()
shape.draw()  // вызов через witness table
```

### Сравнение Swift Runtime и Objective-C Runtime

| Аспект                       | Swift Runtime                                       | Objective-C Runtime                          |
| ---------------------------- | --------------------------------------------------- | -------------------------------------------- |
| Основной механизм            | Value Witness Table, Protocol Witness Table, VTable | objc_msgSend + dispatch table                |
| Скорость                     | Очень быстрая (статическая + динамическая)          | Медленнее (всегда динамическая)              |
| Поддержка value types        | Полная (struct, enum, generics)                     | Отсутствует                                  |
| Рефлексия                    | `Mirror`, `Reflection`                              | `class_getInstanceMethod`, `objc_property_t` |
| [[Swizzling]]                | Сложно (требует `@objc`)                            | Легко (`method_exchangeImplementations`)     |
| Использование в чистом Swift | Основной                                            | Только для ObjC-интеропа                     |

### Когда Swift Runtime активно работает в 2026 году

- `any Protocol` и `some Protocol`
- `Mirror` и рефлексия
- `as?`, `as!`, `is`
- Generics с протоколами
- Динамические вызовы в классах без `final`
- Обработка ошибок ([[throws]], [[try]])
- `type(of:)`, [[MemoryLayout]], [[Unsafe]] [[API]]

### Итог — коротко и честно

- **Swift Runtime** — это **нативная** динамика Swift (с 2019+)
- Он **быстрее** и **мощнее** ObjC Runtime для value types и generics
- ObjC Runtime всё ещё живёт для совместимости с Cocoa/UIKit/AppKit
- В чистом Swift-коде ты работаешь почти всегда с **Swift Runtime**
- Самые частые места, где ты «видишь» его работу:  
  - `Mirror`  
  - `any` / `some` протоколы  
  - `as?` / `as!`  
  - вызовы методов классов без `final`

**Главное правило**:
> «Если ты пишешь чистый Swift без `@objc` и [[Foundation]] — ты работаешь с **Swift Runtime**.  
> Если используешь [[UIKit]]/AppKit/ObjC API — под капотом включается **Objective-C Runtime**.»
