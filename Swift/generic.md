**Generics** — это способ писать **один и тот же код**, который работает **с любым типом данных**, сохраняя при этом **полную типобезопасность** на этапе компиляции.

### 1. Зачем нужны Generics (мотивация)

Без generics пришлось бы писать отдельные функции/структуры для каждого типа:

```swift
func swapInts(_ a: inout Int, _ b: inout Int) { ... }
func swapStrings(_ a: inout String, _ b: inout String) { ... }
func swapDoubles(_ a: inout Double, _ b: inout Double) { ... }
// и так бесконечно...
```

С generics — **одна** функция для всех типов:

```swift
func swap<T>(_ a: inout T, _ b: inout T) {
    (a, b) = (b, a)
}
```

### 2. Базовый синтаксис и примеры

#### 2.1 Generic функция

```swift
func identity<T>(_ value: T) -> T {
    return value
}

print(identity(42))           // 42
print(identity("Hello"))      // "Hello"
print(identity([1,2,3]))      // [1,2,3]
```

#### 2.2 Generic структура (самый частый случай)

```swift
struct Stack<Element> {
    private var storage: [Element] = []
    
    mutating func push(_ element: Element) {
        storage.append(element)
    }
    
    mutating func pop() -> Element? {
        storage.popLast()
    }
    
    var isEmpty: Bool { storage.isEmpty }
    var count: Int { storage.count }
}

// Использование
var intStack = Stack<Int>()
intStack.push(10)
intStack.push(20)
print(intStack.pop()!)     // 20

var stringStack = Stack<String>()
stringStack.push("Swift")
stringStack.push("Rocks")
```

#### 2.3 Generic класс

```swift
class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func set(_ value: Value, for key: Key) {
        storage[key] = value
    }
    
    func get(for key: Key) -> Value? {
        storage[key]
    }
    
    func clear() {
        storage.removeAll()
    }
}

let userCache = Cache<String, User>()
userCache.set(User(name: "Анна"), for: "anna123")
```

#### 2.4 Generic перечисление

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}

let response: Result<String, NetworkError> = .success("Данные получены")
```

### 3. Ограничения типов (Type Constraints)

Без ограничений generic-функция может работать только с операциями, общими для всех типов (присваивание, возврат).

С ограничениями — можно использовать методы и свойства протоколов.

| Ограничение                        | Что даёт                           | Пример использования         |
| ---------------------------------- | ---------------------------------- | ---------------------------- |
| `T:` [[Equatable]]                 | `==`, `!=`                         | `firstIndex(of:)`            |
| `T:` [[Hashable]]                  | `Hashable`, ключи в [[Dictionary]] | `Set<T>`, `Dictionary<T, V>` |
| `T:` [[Comparable]]                | `<`, `>`, `<=`, `>=`, `sorted()`   | Сортировка                   |
| `T:` [[Numeric]]                   | `+`, `-`, `*`, `/`                 | Математика                   |
| `T:` [[Decodable]] & [[Encodable]] | Сериализация ([[Codable]])         | API-модели                   |
| `T:` [[AnyObject]]                 | Только классы ([[Reference Type]]) | `weak var`                   |

Примеры:

```swift
// Только типы, которые можно сравнивать
func contains<T: Equatable>(_ array: [T], _ element: T) -> Bool {
    array.contains(element)
}

// Только числовые типы
func sum<T: Numeric>(_ numbers: [T]) -> T {
    numbers.reduce(0, +)
}

// Только типы, которые можно закодировать
func saveToJSON<T: Encodable>(_ value: T) throws -> Data {
    try JSONEncoder().encode(value)
}
```

### 4. Associated Types в протоколах

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(index: Int) -> Item { get }
}

struct IntArray: Container {
    typealias Item = Int          // можно явно указать
    private var items: [Int] = []
    
    mutating func append(_ item: Int) { items.append(item) }
    var count: Int { items.count }
    subscript(index: Int) -> Int { items[index] }
}
```

### 5. Opaque Types ([[some]]) vs Existential Types ([[any]])

| Конструкция     | Что это                          | Диспетчеризация | Когда использовать                     |
|----------------|----------------------------------|------------------|----------------------------------------|
| `some Protocol`| Конкретный тип, скрытый от вызывающего | Статическая      | Возвращаемый тип функции (быстрее)     |
| `any Protocol` | Экзистенциал (любой тип)         | Динамическая     | Хранение в коллекции, свойствах        |

Пример:

```swift
func makeView() -> some View {      // some → конкретный тип, статическая диспетчеризация
    Text("Hello")
}

let views: [any View] = [...]       // any → разные типы, динамическая диспетчеризация
```

### 6. Реальные сценарии из [[iOS]]-разработки

#### Пример 1 — Универсальный [[API]]-клиент

```swift
struct APIClient {
    func fetch<T: Decodable>(_ type: T.Type, from url: URL) async throws -> T {
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(T.self, from: data)
    }
}

let client = APIClient()
let posts = try await client.fetch([Post].self, from: postsURL)
let user  = try await client.fetch(User.self, from: userURL)
```

#### Пример 2 — Generic ViewModel

```swift
class ListViewModel<Item: Identifiable & Hashable>: ObservableObject {
    @Published var items: [Item] = []
    @Published var isLoading = false
    
    func load() async {
        isLoading = true
        // ...
        items = await fetchItems()
        isLoading = false
    }
}
```

#### Пример 3 — Generic Repository

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

### 7. Типичные ошибки и ловушки

- Забыли указать constraint → ошибка компиляции при попытке использовать специфичные методы
- Использование `any` вместо `some` в возвращаемом типе → потеря производительности
- Слишком много generic-параметров → ухудшение читаемости
- Неправильное использование [[AssociatedType]] → "protocol can only be used as generic constraint"

### 8. Таблица сравнения: когда использовать generics

| Задача                                   | Использовать Generics? | Альтернатива без generics          |
| ---------------------------------------- | ---------------------- | ---------------------------------- |
| Универсальный контейнер                  | Да                     | Отдельные классы для каждого типа  |
| Алгоритм ([[sort]], [[filter]], [[map]]) | Да                     | Дублирование кода                  |
| [[API]]-клиент с разными моделями        | Да                     | Отдельные функции для каждого типа |
| ViewModel для разных сущностей           | Да                     | Множество похожих классов          |
| Протокол с разными типами элементов      | Да (associatedtype)    | Any / протоколы без типа           |

### Итог — золотые правила Generics 2026

1. Хочешь писать **один раз** код, который работает с **разными типами** → используй `<T>`
2. Нужны конкретные возможности типа → добавляй **constraint** (`T: Equatable`, `T: View`, `T: Decodable & Encodable`)
3. Протокол должен работать с разными типами → используй **associatedtype**
4. Возвращаешь протокол из функции → по умолчанию пиши **[[some Protocol]]** (быстрее)
5. Хранишь коллекцию разных типов → используй **[[any Protocol]]**
6. В 95% случаев generics — это **структуры**, **функции** и **протоколы с associatedtype**

**Короткий девиз**:
> «Generics — это когда ты пишешь код один раз, а используешь его миллион раз с разными типами, и компилятор всё проверяет за тебя.»
