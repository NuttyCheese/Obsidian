**Generics** (обобщённые типы / дженерики) — один из самых мощных и часто недооценённых механизмов в [[Swift]].  
Они позволяют писать **один и тот же код**, который работает **с любыми типами**, сохраняя при этом **полную типобезопасность** на этапе компиляции.

> Проще говоря: generics = «одна функция/структура/протокол для всех типов, и компилятор сам проверяет корректность».

### 1. Зачем нужны generics (мотивация 2026)

Без generics пришлось бы дублировать код для каждого типа:

```swift
func printInt(_ value: Int) { print(value) }
func printString(_ value: String) { print(value) }
func printDouble(_ value: Double) { print(value) }
// ... и так для каждого типа
```

С generics — **одна** функция:

```swift
func printValue<T>(_ value: T) {
    print(value)
}

printValue(42)        // 42
printValue("Hello")   // Hello
printValue(3.14)      // 3.14
printValue([1, 2, 3]) // [1, 2, 3]
```

**Преимущества generics**:
- Нет дублирования кода ([[DRY]])
- Полная типобезопасность (компилятор ловит ошибки)
- Максимальная производительность (статическая диспетчеризация)
- Универсальность (один код → миллион типов)

### 2. Базовый синтаксис и примеры

#### 2.1 Generic функция (самый простой случай)

```swift
func swap<T>(_ a: inout T, _ b: inout T) {
    (a, b) = (b, a)
}

var x = 5
var y = 10
swap(&x, &y)
print(x, y) // 10 5

var name1 = "Alice"
var name2 = "Bob"
swap(&name1, &name2)
print(name1, name2) // Bob Alice
```

#### 2.2 Generic структура (самый частый паттерн)

```swift
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        items.popLast()
    }
    
    var isEmpty: Bool { items.isEmpty }
    var count: Int { items.count }
}

var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
print(intStack.pop()!) // 2

var stringStack = Stack<String>()
stringStack.push("Swift")
stringStack.push("Rocks")
```

#### 2.3 Generic класс

```swift
final class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func set(_ value: Value, for key: Key) {
        storage[key] = value
    }
    
    func get(for key: Key) -> Value? {
        storage[key]
    }
    
    func removeAll() {
        storage.removeAll()
    }
}

let userCache = Cache<String, User>()
userCache.set(User(name: "Алекс"), for: "user123")
```

#### 2.4 Generic enum (Result, NetworkResponse и т.д.)

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
    
    var value: Success? {
        if case .success(let v) = self { return v }
        return nil
    }
    
    var error: Failure? {
        if case .failure(let e) = self { return e }
        return nil
    }
}
```

### 3. Ограничения типов ([[Constraint]]s) — ключ к мощности

Без ограничений generic может работать только с операциями, общими для всех типов (присваивание, возврат).

С ограничениями — можно использовать методы и свойства протоколов.

| Ограничение                        | Что позволяет                                | Типичный пример использования            |
| ---------------------------------- | -------------------------------------------- | ---------------------------------------- |
| `T:` [[Equatable]]                 | `==`, `!=`, `contains`, `firstIndex(of:)`    | `contains(_:)`, `firstIndex(of:)`        |
| `T:` [[Hashable]]                  | `Hashable`, ключи в `Dictionary`, `Set`      | [[Set]]`<T>`, [[Dictionary]]`<T, V>`     |
| `T:` [[Comparable]]                | `<`, `>`, `<=`, `>=`, `sorted()`             | `sorted()`, `min()`, `max()`             |
| `T:` [[Numeric]]                   | `+`, `-`, `*`, `/`, `%`                      | `sum`, `average`, математические функции |
| `T:` [[Decodable]] & [[Encodable]] | Сериализация ([[JSON]], PropertyList и т.д.) | [[Codable]] модели в [[API]]-клиенте     |
| `T: View`                          | [[SwiftUI]]-компоненты                       | `some View`, generic view-модификаторы   |
| `T:` [[AnyObject]]                 | Только классы ([[reference type]]s)          | `weak var delegate: T?`                  |
| `T:` [[Sendable]]                  | Безопасность в concurrent коде (Swift 6+)    | [[actor]], [[Task]], [[async let]]       |

Примеры:

```swift
// Только сравнимые типы
func firstIndex<T: Equatable>(of element: T, in array: [T]) -> Int? {
    array.firstIndex(of: element)
}

// Только числовые типы
func average<T: FloatingPoint>(_ numbers: [T]) -> T {
    numbers.reduce(0, +) / T(numbers.count)
}
```

### 4. [[AssociatedType]]s в протоколах (самый мощный паттерн)

```swift
protocol Repository {
    associatedtype Model: Identifiable
    func getAll() async throws -> [Model]
    func get(id: Model.ID) async throws -> Model
}

actor MemoryRepository<Model: Identifiable>: Repository {
    private var storage: [Model.ID: Model] = [:]
    
    func getAll() async throws -> [Model] {
        Array(storage.values)
    }
    
    func get(id: Model.ID) async throws -> Model {
        guard let model = storage[id] else {
            throw RepositoryError.notFound
        }
        return model
    }
}
```

### 5. Opaque Types ([[some]]) vs [[Existential Type]]s ([[any]])

| Конструкция       | Что это                                | Диспетчеризация | Когда использовать в 2026                                     |
| ----------------- | -------------------------------------- | --------------- | ------------------------------------------------------------- |
| [[some Protocol]] | Конкретный тип, скрытый от вызывающего | Статическая     | Возвращаемый тип функции (быстрее, предпочтительнее)          |
| [[any Protocol]]  | Экзистенциал (любой тип)               | Динамическая    | Хранение в коллекции, свойствах, когда тип заранее неизвестен |

```swift
func makeView() -> some View {          // some — статическая диспетчеризация
    Text("Hello")
}

let views: [any View] = [...]           // any — разные типы, динамическая диспетчеризация
```

**Правило 2026**:  
- Возвращаешь протокол из функции → **по умолчанию `some`**  
- Хранишь коллекцию разных типов → **только `any`**

### 6. Лучшие практики generics в Swift 2026

- **По умолчанию** используй generics для контейнеров, алгоритмов, репозиториев, ViewModel  
- Добавляй **ограничения** (`T: Equatable`, `T: View`, `T: Decodable`) — это даёт компилятору больше информации  
- Предпочитай **`some`** над `any` в возвращаемых типах (производительность + безопасность)  
- Делай generic-типы **`Sendable`** (Swift 6) — особенно для concurrent кода  
- Не бойся [[generic]] — они **не замедляют** код в релизе (-O), а часто ускоряют  
- Документируйте — пиши комментарий «<Element: Identifiable> — элементы списка с уникальным ID»

**Короткий девиз 2026**:
> Generics — это когда ты пишешь код **один раз**, а используешь его **миллион раз** с разными типами, и компилятор всё проверяет за тебя.  
> В 2026 году:  
> - `<T: Constraint>` — твой лучший друг  
> - `some` для возвращаемых типов  
> - `any` только для коллекций и свойств  
> Это **основа** современного, типобезопасного и производительного Swift-кода.
