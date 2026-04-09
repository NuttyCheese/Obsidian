#swift #protocols #pop #generics #associatedtype #any #some

---
### Определение
**Протокол** — это **контракт** (blueprint), который определяет минимальный набор требований: свойства, методы, инициализаторы или ассоциированные типы. Тип (struct, class, перечисление) считается **соответствующим протоколу** (conforming), если он реализует все эти требования.

В отличие от классов, протоколы **не хранят состояние** и **не имеют реализации** (кроме реализации по умолчанию в расширениях). Протоколы — краеугольный камень **Protocol-Oriented Programming (POP)**, который Apple продвигает как альтернативу глубокому наследованию классов.

### Зачем нужны протоколы (мотивация)
Без протоколов код был бы привязан к конкретным типам, что ведёт к дублированию и жёсткости.

```swift
// ❌ Без протокола — дублирование
func drawCircle(_ circle: Circle) { circle.draw() }
func drawSquare(_ square: Square) { square.draw() }

// ✅ С протоколом — один код для всех
protocol Drawable { func draw() }

func draw(_ shape: Drawable) { shape.draw() }
```

### Основные термины и возможности

| Термин / Конструкция | Описание | Пример использования |
|----------------------|----------|----------------------|
| **Conformance** | Соответствие типа требованиям протокола | `struct Circle: Drawable` |
| **Property Requirement** | Требование свойства (геттер или геттер+сеттер) | `var area: Double { get }` |
| **Method Requirement** | Требование метода с сигнатурой | `func draw()` |
| **Initializer Requirement** | Требование инициализатора | `init(radius: Double)` |
| **AssociatedType** | Дженерик-заполнитель в протоколе | `associatedtype Element` |
| **Protocol Extension** | Реализация по умолчанию через `extension` | `extension Drawable { func log() {} }` |
| **Optional Requirement** | Необязательные методы (только `@objc` протоколы) | `@objc optional func optionalMethod()` |
| **Class-only Protocol** | Протокол только для классов (`AnyObject`) | `protocol Delegate: AnyObject` |
| **Composition** | Комбинирование протоколов | `typealias View = Drawable & Loggable` |
| **Existential Type (`any`)** | Хранение разных типов, реализующих протокол | `let shapes: [any Drawable]` |
| **Opaque Type (`some`)** | Конкретный, но скрытый тип (статическая диспетчеризация) | `func make() -> some Drawable` |
| **Primary Associated Types** | Явное именование ассоциированных типов (Swift 6) | `protocol Collection<Element>` |

---

## 1. Базовый синтаксис и примеры

### Пример 1 — Простейший протокол с методом

```swift
protocol Greetable {
    func greet() -> String
}

struct Person: Greetable {
    let name: String
    func greet() -> String { "Привет, я \(name)!" }
}

let alice = Person(name: "Алиса")
print(alice.greet()) // "Привет, я Алиса!"
```

### Пример 2 — Требования к свойствам

```swift
protocol Identifiable {
    var id: String { get }          // только чтение
    var mutableID: String { get set } // чтение и запись
}

struct User: Identifiable {
    let id: String
    var mutableID: String
}

let user = User(id: "u123", mutableID: "temp")
print(user.id)         // "u123"
user.mutableID = "new" // ✅ Можно изменить
```

### Пример 3 — Требование инициализатора

```swift
protocol Creatable {
    init(name: String)
}

class Animal: Creatable {
    let name: String
    required init(name: String) { // `required` обязательно для классов
        self.name = name
    }
}
```

---

## 2. Реализация по умолчанию (Protocol Extensions)

Расширения позволяют добавлять реализацию методов и вычисляемых свойств, что даёт **множественное наследование поведения** (в отличие от классов).

```swift
protocol Loggable {
    var logMessage: String { get }
}

extension Loggable {
    var logMessage: String { "Log: \(self)" }
    func log() { print(logMessage) }
}

struct Product: Loggable { let name: String }
let phone = Product(name: "iPhone")
phone.log() // "Log: Product(name: \"iPhone\")"
```

### Условная реализация (where)

```swift
extension Array: Loggable where Element: CustomStringConvertible {
    var logMessage: String { "Array: \(self)" }
}
```

---

## 3. Associated Types (Дженерики для протоколов)

`associatedtype` — это заполнитель для конкретного типа, который определяется реализующим типом.

```swift
protocol Container {
    associatedtype Item
    mutating func add(_ item: Item)
    var count: Int { get }
}

struct IntBox: Container {
    typealias Item = Int  // можно явно
    private var items: [Int] = []
    mutating func add(_ item: Int) { items.append(item) }
    var count: Int { items.count }
}

struct GenericBox<T>: Container {
    typealias Item = T
    private var items: [T] = []
    mutating func add(_ item: T) { items.append(item) }
    var count: Int { items.count }
}
```

### Primary Associated Types (Swift 6)

```swift
protocol Collection<Element> {
    associatedtype Element
    associatedtype Index
}
```

---

## 4. Self и ограничения (where)

`Self` ссылается на конкретный тип, реализующий протокол.

```swift
protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}

protocol Cloneable {
    func clone() -> Self  // возвращает тот же тип
}

struct Point: Cloneable {
    var x, y: Int
    func clone() -> Self { Point(x: x, y: y) }
}
```

### Ограничения через `where`

```swift
extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        // доступно только для Equatable Item
        return false
    }
}
```

---

## 5. Протоколы как типы

### Existential Type (`any`)

Позволяет хранить **разные типы**, реализующие протокол. Использует **динамическую диспетчеризацию** (witness table).

```swift
let shapes: [any Drawable] = [Circle(), Square()]
for shape in shapes { shape.draw() }
```

### Opaque Type (`some`)

Возвращает **конкретный, но скрытый тип**. Использует **статическую диспетчеризацию** (быстрее).

```swift
func makeDrawable() -> some Drawable {
    return Circle() // тип известен компилятору
}
```

---

## 6. Протоколы и классы (Class-Only, Weak, Delegate)

```swift
protocol Delegate: AnyObject {
    func didFinish()
}

class Worker {
    weak var delegate: Delegate? // только классы, weak разрешён
}
```

---

## 7. Protocol Composition (Комбинирование)

```swift
typealias NamedAged = Named & Aged

struct User: NamedAged {
    let name: String
    let age: Int
}
```

---

## 8. Optional Requirements (@objc)

Только для протоколов, видимых в Objective-C.

```swift
@objc protocol OptionalDelegate {
    @objc optional func optionalMethod()
}
```

---

## 9. Реальные сценарии использования

### Repository Pattern

```swift
protocol Repository {
    associatedtype Model: Identifiable
    func getAll() async throws -> [Model]
    func get(id: Model.ID) async throws -> Model
}
```

### Coordinator / Router

```swift
protocol Router {
    associatedtype Route: Hashable
    func navigate(to route: Route)
}
```

### ViewModel для SwiftUI

```swift
protocol ViewModel: ObservableObject {
    associatedtype State
    var state: State { get }
    func load()
}
```

---

## 10. Типичные ошибки и ловушки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| Забыли `@objc dynamic` | KVO не работает | Добавить `@objc dynamic` |
| `any` вместо `some` | Потеря производительности | Использовать `some` где тип фиксирован |
| Слишком много `associatedtype` | Сложность | Декомпозиция протоколов |
| `init` без `required` в классе | Наследники не смогут соответствовать | Добавить `required` |
| Optional requirement без `@objc` | Ошибка компиляции | Добавить `@objc` |

---

## 11. Итог — золотые правила протоколов

1. **Абстракция** → протокол.
2. **Реализация по умолчанию** → `extension Protocol`.
3. **Дженерик-протокол** → `associatedtype`.
4. **Фиксированный тип в возврате** → `some Protocol` (быстрее).
5. **Коллекция разных типов** → `any Protocol`.
6. **Работа с UIKit/ObjC** → добавлять `@objc` и `AnyObject`.
7. **Условная функциональность** → `where`.

---

**Короткий девиз**:
> «Протокол — это контракт.  
> Чем больше контрактов — тем меньше зависимостей и тем легче менять, тестировать и масштабировать код.»