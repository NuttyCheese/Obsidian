Протокол `Hashable` в [[Swift]] позволяет типу участвовать в хэш-вычислениях. Типы, соответствующие этому протоколу, можно использовать в коллекциях, таких как [[Set]] и в качестве ключей для [[Dictionary]]. Это особенно полезно для оптимизации поиска и проверки на уникальность.

---

### **Что такое `Hashable`?**

`Hashable` — это протокол, который требует от типа реализации метода для вычисления хэш-значения. Хэш-значение используется для сравнения объектов на равенство с высокой производительностью.

---

### **Объявление протокола**

```swift
protocol Hashable: Equatable {
    func hash(into hasher: inout Hasher)
}
```

- **Наследование от [[Equatable]]**: Типы, соответствующие `Hashable`, также обязаны реализовывать метод сравнения `==`.
- **Метод `hash(into:)`**: Используется для предоставления информации, необходимой для вычисления хэш-значения.

---

### **Когда нужен `Hashable`?**

1. **Использование в коллекциях:**
    
    - Хэшируемые типы можно хранить в `Set` или использовать в качестве ключей в `Dictionary`.
2. **Уникальность объектов:**
    
    - `Hashable` позволяет быстро проверять, содержится ли объект в коллекции.
3. **Оптимизация поиска:**
    
    - Хэш-значения позволяют организовать быстрый поиск.

---

### **Пример простейшего использования**

#### **Пример с `Set`**

```swift
struct Person: Hashable {
    let name: String
    let age: Int
}

let person1 = Person(name: "Alice", age: 30)
let person2 = Person(name: "Bob", age: 25)
let person3 = Person(name: "Alice", age: 30)

var peopleSet: Set<Person> = [person1, person2]

// Проверка уникальности
print(peopleSet.contains(person3)) // true, так как person1 и person3 равны
```

---

### **Как Swift автоматически реализует `Hashable`**

Swift автоматически предоставляет реализацию `Hashable` для структур и перечислений, если все их свойства соответствуют `Hashable`.

```swift
struct User: Hashable {
    let id: Int
    let username: String
}

let user1 = User(id: 1, username: "john_doe")
let user2 = User(id: 2, username: "jane_doe")
```

Здесь Swift сам реализует `hash(into:)` и `==`, так как все свойства `id` и `username` соответствуют `Hashable`.

---

### **Ручная реализация `Hashable`**

Если нужно более детально управлять процессом хэширования, можно вручную реализовать `hash(into:)`.

```swift
struct CustomPerson: Hashable {
    let name: String
    let age: Int

    func hash(into hasher: inout Hasher) {
        hasher.combine(name)
        hasher.combine(age)
    }

    static func == (lhs: CustomPerson, rhs: CustomPerson) -> Bool {
        return lhs.name == rhs.name && lhs.age == rhs.age
    }
}
```

- **`hasher.combine(_:)`**: Используется для добавления свойств в процесс хэширования.
- Реализация `==` проверяет равенство по всем необходимым полям.

---

### **Пользовательский хэш для оптимизации**

Можно управлять приоритетами полей в хэшировании. Например, использовать только уникальный идентификатор:

```swift
struct Employee: Hashable {
    let id: Int
    let name: String

    func hash(into hasher: inout Hasher) {
        hasher.combine(id) // Только ID влияет на хэш
    }

    static func == (lhs: Employee, rhs: Employee) -> Bool {
        return lhs.id == rhs.id // Только ID влияет на сравнение
    }
}

let emp1 = Employee(id: 1, name: "John")
let emp2 = Employee(id: 1, name: "Johnny")

print(emp1 == emp2) // true, так как ID совпадают
```

---

### **Использование перечислений (`enum`) с `Hashable`**

Перечисления автоматически соответствуют `Hashable`, если их ассоциативные значения также соответствуют `Hashable`.

```swift
enum Direction: Hashable {
    case north
    case south
    case east
    case west
}

let directionSet: Set<Direction> = [.north, .south]
print(directionSet.contains(.west)) // false
```

С ассоциативными значениями:

```swift
enum Shape: Hashable {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
}

let shape1 = Shape.circle(radius: 5)
let shape2 = Shape.rectangle(width: 10, height: 20)

var shapeSet: Set<Shape> = [shape1, shape2]
print(shapeSet.contains(.circle(radius: 5))) // true
```

---

### **Хэширование и производительность**

- **Хороший хэш-функционал**:
    
    - Уменьшает вероятность коллизий.
    - Улучшает производительность операций `Set` и `Dictionary`.
- **Плохой хэш-функционал**:
    
    - Может привести к большому количеству коллизий, ухудшая производительность.

Пример плохой реализации:

```swift
struct BadHashPerson: Hashable {
    let name: String
    let age: Int

    func hash(into hasher: inout Hasher) {
        hasher.combine(0) // Плохой хэш — все объекты будут иметь одинаковое значение
    }
}
```

---

### **Работа с `Dictionary`**

Ключи словаря должны быть `Hashable`.

```swift
struct Point: Hashable {
    let x: Int
    let y: Int
}

var points: [Point: String] = [
    Point(x: 0, y: 0): "Origin",
    Point(x: 1, y: 1): "First Quadrant"
]

print(points[Point(x: 0, y: 0)] ?? "Not found") // "Origin"
```

---

### **Заключение**

Протокол `Hashable` — это основа для многих коллекций в Swift. Его использование обеспечивает:

1. **Уникальность объектов.**
2. **Оптимизацию поиска и хранения.**
3. **Гибкость в определении хэширования.**

Используйте автоматическую реализацию `Hashable`, если она подходит, а для сложных типов создавайте пользовательскую реализацию, чтобы гарантировать правильное поведение и производительность.