**`associatedtype`** — это placeholder (заполнитель) типа, который **определяется конкретным типом при реализации протокола**.

- Используется только внутри протоколов
    
- Позволяет протоколам быть **обобщёнными** без необходимости использовать [[generic]] на каждом методе
    
- Тип задаётся **реализующим классом/структурой/[[enum]]**
    

> Проще говоря: Associated Type = «тип, который будет определён позже, когда протокол реализуется».

---

## 2. Основные термины

| Термин                  | Описание                                                             |
| ----------------------- | -------------------------------------------------------------------- |
| **[[Protocol]]**        | Интерфейс, который может использовать `associatedtype`               |
| **Generic placeholder** | Место, где будет конкретный тип                                      |
| **Concrete Type**       | Конкретный тип, который заменяет `associatedtype` при реализации     |
| **Constraints**         | Ограничения на associated type (`associatedtype Element: Equatable`) |

---

## 3. Основной синтаксис

```swift
protocol Container {
    associatedtype Item
    var items: [Item] { get set }
    mutating func append(_ item: Item)
}
```

- `Item` — это **associated type**, который заменится на конкретный тип при реализации
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший протокол с Associated Type

```swift
protocol Container {
    associatedtype Item
    var items: [Item] { get set }
    mutating func append(_ item: Item)
}

struct IntContainer: Container {
    var items = [Int]()
    mutating func append(_ item: Int) {
        items.append(item)
    }
}

var c = IntContainer()
c.append(5)
print(c.items) // [5]
```

- `Item` заменён на [[Int]]
    

---

### Пример 2. Протокол с generic constraints

```swift
protocol SummableContainer {
    associatedtype Element: Numeric
    var items: [Element] { get set }
    func sum() -> Element
}

struct IntArray: SummableContainer {
    var items: [Int]
    func sum() -> Int {
        return items.reduce(0, +)
    }
}

let arr = IntArray(items: [1, 2, 3])
print(arr.sum()) // 6
```

- `Element: Numeric` → ограничение на тип
    
- Позволяет использовать операции `+`, `*` и т.д.
    

---

### Пример 3. Associated Type с классом

```swift
protocol DataSource {
    associatedtype DataType
    func fetchData() -> DataType
}

class StringFetcher: DataSource {
    func fetchData() -> String {
        return "Hello"
    }
}

let fetcher = StringFetcher()
print(fetcher.fetchData()) // Hello
```

- `DataType` заменён на [[String]]
    

---

### Пример 4. Associated Type с массивом и функцией

```swift
protocol StackProtocol {
    associatedtype Element
    mutating func push(_ item: Element)
    mutating func pop() -> Element?
}

struct IntStack: StackProtocol {
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int? {
        return items.popLast()
    }
}

var stack = IntStack()
stack.push(10)
print(stack.pop()!) // 10
```

- Элемент стека определяется через `associatedtype Element`
    

---

### Пример 5. Использование typealias для clarity

```swift
protocol Repository {
    associatedtype Model
    func save(_ item: Model)
}

struct User { var name: String }

struct UserRepository: Repository {
    typealias Model = User
    func save(_ item: User) {
        print("Saving user \(item.name)")
    }
}

let repo = UserRepository()
repo.save(User(name: "Alice"))
```

- `typealias` помогает явно указать конкретный тип для `associatedtype`
    

---

## 5. Особенности Associated Type

1. Используется **только внутри протоколов**
    
2. Позволяет делать **обобщённые протоколы**
    
3. Тип определяется **при реализации протокола**
    
4. Можно задавать **constraints** (`associatedtype Item: Equatable`)
    
5. Можно комбинировать с **[[typealias]]** для явного указания типа
    

---

## 6. Итог

- **Associated Type** = placeholder типа в протоколе
    
- Используется для **обобщённых и гибких протоколов**
    
- Позволяет создавать **контейнеры, стеки, репозитории и др. структуры**
    
- Конкретный тип определяется **реализующей структурой/классом**
    
- Улучшает **читаемость и переиспользуемость кода**
    

---
