**Protocol** — это **контракт**, который описывает, **какие свойства, методы, инициализаторы или [[AssociatedType]]** должен реализовать тип, чтобы считаться соответствующим этому протоколу.

Протокол **не содержит реализации** (кроме случаев [[default]] через [[extension]]), он только **требует**.

### 1. Зачем нужны протоколы (мотивация)

Без протоколов пришлось бы работать только с конкретными типами:

```swift
func drawCircle(_ circle: Circle) { ... }
func drawSquare(_ square: Square) { ... }
// и так для каждого типа фигуры
```

С протоколами — один код для всех типов, которые его реализуют:

```swift
protocol Drawable {
    func draw()
}

func drawShape(_ shape: any Drawable) {
    shape.draw()
}
```

### 2. Основные термины и возможности

| Термин / Конструкция           | Описание                                                             | Пример использования                             |
| ------------------------------ | -------------------------------------------------------------------- | ------------------------------------------------ |
| **Conformance**                | Соответствие типа протоколу                                          | `struct Circle: Drawable`                        |
| **Property Requirement**       | Требование свойства ([[get]] / [[get set]])                          | `var area: Double { get }`                       |
| **Method Requirement**         | Требование метода с сигнатурой                                       | `func draw()`                                    |
| **Initializer Requirement**    | Требование инициализатора                                            | `init(radius: Double)`                           |
| **[[AssociatedType]]**         | Замещаемый тип ([[generic]] в протоколе)                             | `associatedtype Element`                         |
| **[[Protocol]] [[Extension]]** | Default-реализация методов/свойств через extension                   | `extension Drawable { func describe() { ... } }` |
| **[[Optional]] Requirement**   | Необязательные методы (только `@objc` протоколы)                     | `@objc optional func optionalMethod()`           |
| **Class-only Protocol**        | Протокол, который могут реализовывать только классы ([[AnyObject]])  | `protocol Delegate: AnyObject { ... }`           |
| **Composition**                | Комбинирование протоколов                                            | `typealias DrawableView = Drawable & View`       |
| **Existential Type** ([[any]]) | Хранение разных типов, реализующих протокол                          | `let shapes: [any Drawable]`                     |
| **Opaque Type** ([[some]])     | Конкретный тип, скрытый от вызывающего (статическая диспетчеризация) | `func makeView() -> some View`                   |

### 3. Базовый синтаксис и примеры (от простого к сложному)

#### Пример 1 — Простейший протокол с методом

```swift
protocol Greetable {
    func greet() -> String
}

struct Person: Greetable {
    let name: String
    
    func greet() -> String {
        "Привет, меня зовут \(name)!"
    }
}

let alice = Person(name: "Алиса")
print(alice.greet())  // Привет, меня зовут Алиса!
```

#### Пример 2 — Протокол со свойством

```swift
protocol Identifiable {
    var id: String { get }          // только чтение
    var mutableID: String { get set } // чтение + запись
}

struct User: Identifiable {
    let id: String
    var mutableID: String
}

let user = User(id: "u123", mutableID: "temp")
print(user.id)         // u123
user.mutableID = "new" // можно изменить
```

#### Пример 3 — Default-реализация через extension

```swift
protocol Loggable {
    var logDescription: String { get }
}

extension Loggable {
    var logDescription: String {
        "Объект: \(type(of: self))"
    }
    
    func log() {
        print(logDescription)
    }
}

struct Product: Loggable {
    let name: String
}

let phone = Product(name: "iPhone")
phone.log()  // Объект: Product
```

#### Пример 4 — Associated Type (самый мощный инструмент)

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(index: Int) -> Item { get }
}

struct IntArray: Container {
    typealias Item = Int
    private var items: [Int] = []
    
    mutating func append(_ item: Int) { items.append(item) }
    var count: Int { items.count }
    subscript(index: Int) -> Int { items[index] }
}

var numbers = IntArray()
numbers.append(10)
numbers.append(20)
print(numbers[1])  // 20
```

#### Пример 5 — Протокол с инициализатором

```swift
protocol Creatable {
    init(name: String)
}

class User: Creatable {
    let name: String
    
    required init(name: String) {  // required — обязательно для наследников
        self.name = name
    }
}

let user = User(name: "Боб")
```

#### Пример 6 — Protocol Composition (комбинирование)

```swift
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

typealias PersonType = Named & Aged

struct Citizen: PersonType {
    let name: String
    let age: Int
}

func describe(person: PersonType) {
    print("\(person.name), возраст: \(person.age)")
}
```

#### Пример 7 — Class-only протокол

```swift
protocol Delegate: AnyObject {
    func didFinishTask()
}

class Worker {
    weak var delegate: Delegate?
}
```

#### Пример 8 — [[Optional]] требования (только [[@objc]] протоколы)

```swift
@objc protocol OptionalDelegate {
    @objc optional func optionalMethod()
}

class MyClass: OptionalDelegate {
    // optionalMethod можно не реализовывать
}
```

#### Пример 9 — Протокол как тип (any vs some)

```swift
func makeLogger() -> some Logger {      // some → конкретный тип, статическая диспетчеризация
    ConsoleLogger()
}

let loggers: [any Logger] = [...]       // any → разные типы, динамическая диспетчеризация
```

### 4. Реальные сценарии в iOS-разработке (2026)

#### Сценарий 1 — Модель данных + [[Codable]]

```swift
protocol Identifiable {
    associatedtype ID
    var id: ID { get }
}

struct Post: Codable, Identifiable {
    typealias ID = Int
    let id: ID
    let title: String
}
```

#### Сценарий 2 — ViewModel для [[UIKit]]/[[SwiftUI]]

```swift
protocol ViewModel: ObservableObject {
    associatedtype State
    var state: State { get }
    func load()
}
```

#### Сценарий 3 — Repository паттерн

```swift
protocol Repository {
    associatedtype Model: Identifiable
    func getAll() async throws -> [Model]
    func get(id: Model.ID) async throws -> Model
}
```

#### Сценарий 4 — [[Coordinator]] / [[Clean Swift (VIP) Architecture#**1. Взаимодействие компонентов (VIP-цикл)**|Router]]

```swift
protocol Router {
    associatedtype Route: Hashable
    func navigate(to route: Route)
}
```

### 5. Типичные ошибки и ловушки

- Забыли `@objc dynamic` — [[KVO]] не работает
- Использовали `any` вместо `some` в возвращаемом типе — потеря производительности
- Слишком много `associatedtype` → сложный код
- Протокол с `init` без `required` — нельзя наследовать
- Optional требования без `@objc` — ошибка компиляции

### 6. Таблица: когда использовать протоколы

| Задача                           | Протокол нужен? | Пример протокола                 | Альтернатива без протокола |
| -------------------------------- | --------------- | -------------------------------- | -------------------------- |
| Общий интерфейс для разных типов | Да              | `Drawable`, `Loggable`           | Конкретные классы          |
| Полиморфизм в коллекции          | Да              | `[any Shape]`                    | Отдельные массивы          |
| Зависимость от абстракции (DI)   | Да              | `NetworkService`                 | Конкретный класс           |
| Универсальный контейнер          | Да              | [[Collection]]                   | —                          |
| Связывание данных и UI (SwiftUI) | Да              | `ObservableObject`, `@Published` | —                          |

### 7. Итог — золотые правила протоколов 2026

1. Хочешь **абстрагироваться** от конкретного типа → пиши протокол
2. Хочешь **default-реализацию** → делай extension
3. Хочешь **generic-протокол** → используй associatedtype
4. Возвращаешь протокол из функции → пиши **`some Protocol`** (быстрее)
5. Хранишь коллекцию разных типов → используй **`any Protocol`**
6. Работаешь с UIKit/ObjC → добавляй `@objc` и `AnyObject` когда нужно
7. В 95% случаев протоколы — это **интерфейсы**, **контракты** и **абстракции**

**Короткий девиз**:
> «Протокол — это контракт.  
> Чем больше контрактов — тем меньше зависимостей и тем легче менять/тестировать/масштабировать код.»
