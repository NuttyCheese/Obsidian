#swift #generics #type-safety #protocols #associatedtype #some #any

---
### Определение
**Generic (дженерики, обобщённое программирование)** — это одна из самых мощных и фундаментальных возможностей [[Swift]], позволяющая писать гибкий, переиспользуемый и типобезопасный код без дублирования . Дженерики позволяют создавать функции, структуры, классы, перечисления и протоколы, которые могут работать с **любым типом**, сохраняя при этом строгую типизацию .

Простыми словами, дженерики — это «шаблоны» кода, где конкретный тип указывается позже, при использовании. Вместо того чтобы писать отдельную функцию для [[Int]], отдельную для [[String]] и отдельную для [[Double]], вы пишете одну обобщённую функцию, которая работает со всеми ними.

### Зачем это знать iOS-разработчику?
1.  **Переиспользование кода:** Один раз написали — используете с разными типами.
2.  **Типобезопасность:** Ошибки типов выявляются на этапе компиляции.
3.  **Стандартная библиотека Swift:** `Array<Element>`, `Optional<Wrapped>`, `Dictionary<Key, Value>` — всё это дженерики.
4.  **Протокол-ориентированное программирование:** Дженерики в сочетании с протоколами дают невероятную гибкость.
5.  **Создание переиспользуемых компонентов:** UI-компоненты, сетевые сервисы, хранилища данных.

---

### Базовый синтаксис

#### 1. **Generic-функции**

```swift
// Функция, которая меняет местами два значения любого типа
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 5
var y = 10
swapValues(&x, &y)
print(x, y)  // 10, 5

var str1 = "Hello"
var str2 = "World"
swapValues(&str1, &str2)
print(str1, str2)  // World, Hello
```

#### 2. **Generic-типы ([[struct]], [[class]], [[enum]])**

```swift
// Generic-структура для стека
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    var count: Int {
        return items.count
    }
}

var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
print(intStack.pop() ?? 0)  // 2

var stringStack = Stack<String>()
stringStack.push("Hello")
stringStack.push("World")
print(stringStack.pop() ?? "")  // World
```

---

### Ограничения ([[Constraint]]s)

Дженерики можно ограничивать, указывая, что тип должен соответствовать определённому протоколу или наследовать от класса.

#### 1. **Ограничение протоколом**

```swift
// Функция работает только с типами, соответствующими Equatable
func findIndex<T: Equatable>(of value: T, in array: [T]) -> Int? {
    for (index, item) in array.enumerated() {
        if item == value {
            return index
        }
    }
    return nil
}

let numbers = [1, 2, 3, 4, 5]
print(findIndex(of: 3, in: numbers) ?? -1)  // 2

let strings = ["a", "b", "c"]
print(findIndex(of: "b", in: strings) ?? -1)  // 1
```

#### 2. **Ограничение классом**

```swift
class Animal { }
class Dog: Animal { }
class Cat: Animal { }

// Функция работает только с Animal или его подклассами
func walk<T: Animal>(_ animal: T) {
    print("\(animal) walks")
}

walk(Dog())  // ✅
walk(Cat())  // ✅
// walk(Int())  // ❌ Ошибка: Int не является Animal
```

#### 3. **Множественные ограничения**

```swift
// Тип должен соответствовать обоим протоколам
func process<T: Equatable & Hashable>(_ value: T) {
    print("Value is equatable and hashable")
}

process(5)       // ✅
process("Hello") // ✅
```

---

### [[where]]-клауза

`where` позволяет добавлять более сложные ограничения.

```swift
// Функция работает с массивами, элементы которых Equatable
func containsDuplicates<T>(_ array: [T]) -> Bool where T: Equatable {
    for i in 0..<array.count {
        for j in (i+1)..<array.count {
            if array[i] == array[j] {
                return true
            }
        }
    }
    return false
}

print(containsDuplicates([1, 2, 3, 1]))  // true
print(containsDuplicates(["a", "b", "c"]))  // false
```

```swift
// Расширение для Dictionary, где Key: Hashable
extension Dictionary where Key: Hashable {
    func keysAsArray() -> [Key] {
        return Array(self.keys)
    }
}

let dict = ["a": 1, "b": 2]
print(dict.keysAsArray())  // ["a", "b"]
```

---

### Generic-протоколы (ассоциированные типы)

Протоколы не могут содержать дженерик-параметры напрямую, но используют **ассоциированные типы** ([[associatedtype]]).

```swift
protocol Container {
    associatedtype Item
    mutating func add(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

// Реализация для Int
struct IntArray: Container {
    typealias Item = Int
    private var items: [Int] = []
    
    mutating func add(_ item: Int) {
        items.append(item)
    }
    
    var count: Int {
        return items.count
    }
    
    subscript(i: Int) -> Int {
        return items[i]
    }
}

// Реализация для String
struct StringArray: Container {
    typealias Item = String
    private var items: [String] = []
    
    mutating func add(_ item: String) {
        items.append(item)
    }
    
    var count: Int {
        return items.count
    }
    
    subscript(i: Int) -> String {
        return items[i]
    }
}
```

#### 2. **Generic-реализация протокола**

```swift
struct GenericContainer<T>: Container {
    typealias Item = T
    private var items: [T] = []
    
    mutating func add(_ item: T) {
        items.append(item)
    }
    
    var count: Int {
        return items.count
    }
    
    subscript(i: Int) -> T {
        return items[i]
    }
}

var container = GenericContainer<Int>()
container.add(1)
container.add(2)
print(container[0])  // 1
```

---

### [[some]] и [[any]] для протоколов

#### 1. **`some` (Opaque Type)**

`some` скрывает конкретный тип, но компилятор знает его и может оптимизировать (статическая диспетчеризация).

```swift
protocol Drawable {
    func draw() -> String
}

struct Circle: Drawable {
    func draw() -> String { return "○" }
}

struct Square: Drawable {
    func draw() -> String { return "□" }
}

// Возвращает конкретный, но скрытый тип
func makeDrawable() -> some Drawable {
    return Circle()  // Тип зафиксирован, но скрыт
}

let shape = makeDrawable()
print(shape.draw())  // ○
```

#### 2. **`any` ([[Existential Type]])**

`any` позволяет хранить **разные типы**, соответствующие протоколу (динамическая диспетчеризация).

```swift
let shapes: [any Drawable] = [Circle(), Square()]

for shape in shapes {
    print(shape.draw())  // ○, □
}
```

#### 3. **Сравнение `some` и `any`**

| Аспект              | [[some Protocol]]                  | [[any Protocol]]                     |
| ------------------- | ---------------------------------- | ------------------------------------ |
| **Тип**             | Один конкретный, скрытый           | Любой тип, соответствующий протоколу |
| **Диспетчеризация** | Статическая (быстро)               | Динамическая (медленнее)             |
| **Массив**          | Нельзя смешивать типы              | Можно смешивать                      |
| **Возврат**         | Идеально для возвращаемых значений | Для коллекций и параметров           |

```swift
// ✅ some — один тип
func makeView() -> some View {
    return Text("Hello")  // Всегда Text
}

// ✅ any — разные типы
let views: [any View] = [Text("A"), Button("B") { }]
```

---

### Associatedtype и `where`

#### 1. **Ограничения для associatedtype**

```swift
protocol Container {
    associatedtype Item: Equatable  // Item должен быть Equatable
    func contains(_ item: Item) -> Bool
}

struct MyContainer<T: Equatable>: Container {
    typealias Item = T
    private var items: [T] = []
    
    func contains(_ item: T) -> Bool {
        return items.contains(item)
    }
}
```

#### 2. **`where` для associatedtype**

```swift
protocol Storage {
    associatedtype Item
    mutating func store(_ item: Item)
    var count: Int { get }
}

extension Storage where Item == String {
    func joined(separator: String = "") -> String {
        // Доступно только для Storage с Item == String
        return ""
    }
}

extension Storage where Item: Numeric {
    func sum() -> Item? {
        // Доступно только для Numeric элементов
        return nil
    }
}
```

---

### Продвинутые примеры

#### 1. **Generic-репозиторий**

```swift
protocol Repository {
    associatedtype Entity: Codable & Identifiable
    func get(id: Entity.ID) async throws -> Entity
    func getAll() async throws -> [Entity]
    func save(_ entity: Entity) async throws
    func delete(id: Entity.ID) async throws
}

class UserRepository: Repository {
    typealias Entity = User
    
    func get(id: UUID) async throws -> User {
        // Реализация
        return User(id: id, name: "Alice")
    }
    
    func getAll() async throws -> [User] {
        // Реализация
        return []
    }
    
    func save(_ entity: User) async throws { }
    func delete(id: UUID) async throws { }
}
```

#### 2. **Generic-сервис с несколькими типами**

```swift
class NetworkService {
    func fetch<T: Decodable>(_ endpoint: String) async throws -> T {
        let url = URL(string: endpoint)!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}

struct User: Decodable {
    let id: Int
    let name: String
}

struct Product: Decodable {
    let id: Int
    let title: String
    let price: Double
}

let service = NetworkService()

Task {
    let user: User = try await service.fetch("https://api.example.com/users/1")
    let products: [Product] = try await service.fetch("https://api.example.com/products")
}
```

#### 3. **Generic-стек с [[Comparable]]**

```swift
struct SortedStack<T: Comparable> {
    private var items: [T] = []
    
    mutating func push(_ item: T) {
        items.append(item)
        items.sort()
    }
    
    mutating func pop() -> T? {
        return items.popLast()
    }
    
    var max: T? {
        return items.last
    }
    
    var min: T? {
        return items.first
    }
}

var stack = SortedStack<Int>()
stack.push(5)
stack.push(1)
stack.push(3)
print(stack.max ?? 0)  // 5
print(stack.min ?? 0)  // 1
```

#### 4. **Generic-паттерн Реестр**

```swift
class Registry<T> {
    private var items: [String: T] = [:]
    
    func register(id: String, value: T) {
        items[id] = value
    }
    
    func get(id: String) -> T? {
        return items[id]
    }
    
    func remove(id: String) {
        items.removeValue(forKey: id)
    }
    
    var allValues: [T] {
        return Array(items.values)
    }
}

// Использование для разных типов
let userRegistry = Registry<User>()
userRegistry.register(id: "user1", value: User(id: 1, name: "Alice"))

let productRegistry = Registry<Product>()
productRegistry.register(id: "prod1", value: Product(id: 1, title: "Phone", price: 999.99))
```

---

### Generic и производительность

| Аспект | Generic | Protocol (any) |
|---|---|---|
| **Диспетчеризация** | Статическая (быстро) | Динамическая (медленнее) |
| **Инлайнинг** | Возможен | Ограничен |
| **Размер кода** | Может увеличиться (специализация) | Меньше |
| **Overhead** | Минимальный | Existential container |

```swift
// Быстро (статическая диспетчеризация)
func drawShape<T: Drawable>(_ shape: T) {
    shape.draw()
}

// Медленнее (динамическая через witness table)
func drawShape(_ shape: any Drawable) {
    shape.draw()
}
```

---

### Распространённые ошибки

#### 1. **Использование generic как `any`**

```swift
// ❌ Ошибка: Protocol 'Container' can only be used as a generic constraint
func process(_ container: Container) { }

// ✅ Правильно: через generic constraint
func process<T: Container>(_ container: T) { }
```

#### 2. **Смешивание `some` и `any`**

```swift
// ❌ Нельзя смешивать типы в массиве some
let shapes: [some Drawable] = [Circle(), Square()]  // Ошибка

// ✅ any позволяет смешивать
let shapes: [any Drawable] = [Circle(), Square()]
```

#### 3. **Ассоциированный тип без ограничений**

```swift
protocol Container {
    associatedtype Item  // Без ограничений
}

// ❌ Нельзя использовать как тип
func process(_ container: Container) { }  // Ошибка

// ✅ Нужно ограничение
func process<T: Container>(_ container: T) { }
```

---

### Лучшие практики

| Практика | Почему |
|---|---|
| **Используйте осмысленные имена** | `T`, `U` — для коротких функций, `Element`, `Key`, `Value` — для сложных |
| **Добавляйте ограничения** | Не делайте дженерики "слишком общими" |
| **Предпочитайте generic `some` для возврата** | Быстрее, чем `any` |
| **Используйте `any` для коллекций** | Когда нужны разные типы |
| **Не злоупотребляйте дженериками** | Иногда конкретный тип проще и понятнее |

---

### Короткое правило

> **Generic** = код, работающий с любым типом, сохраняя типобезопасность.  
> **`<T>`** — placeholder для типа.  
> **`where`** — добавляет условия.  
> **`some`** — скрытый конкретный тип (быстро).  
> **`any`** — динамический протокольный тип (медленнее, но гибче).

---

### Итог

**Generic** в Swift:

| Аспект | Значение |
|---|---|
| **Назначение** | Переиспользуемый, типобезопасный код |
| **Где используется** | Функции, структуры, классы, перечисления, протоколы |
| **Ограничения** | `T: Protocol`, `where` |
| **Ассоциированные типы** | Для протоколов (`associatedtype`) |
| **`some` vs `any`** | `some` — один тип (быстро), `any` — разные типы (медленнее) |
| **Производительность** | Высокая (статическая диспетчеризация) |

**Главное правило:**
> Используй дженерики, чтобы избежать дублирования кода. Добавляй ограничения (`where`), чтобы сохранить типобезопасность. Для возвращаемых значений предпочитай `some`, для коллекций разных типов — `any`. Дженерики — фундамент стандартной библиотеки Swift (`Array`, `Optional`, `Dictionary`). Они также необходимы для протокол-ориентированного программирования и создания переиспользуемых компонентов. Помни, что дженерики не должны усложнять код — если дженерик делает код менее читаемым, возможно, лучше использовать конкретный тип.