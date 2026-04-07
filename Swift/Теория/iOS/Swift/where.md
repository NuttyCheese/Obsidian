#swift #where #generics #pattern-matching #constraints #switch #for-in #do-catch

---
### Определение
**`where`** — это ключевое слово в [[Swift]], используемое для добавления дополнительных условий и ограничений в различных контекстах. Оно позволяет уточнять, фильтровать или ограничивать:

- **Generic constraints** (ограничения дженериков)
- **Pattern matching** (сопоставление с образцом в `switch`, `if case`, `guard case`, `for-in`)
- **Do-catch** (фильтрация ошибок)
- **[[Protocol]] [[AssociatedType]]s** (ограничения ассоциированных типов)

`where` делает код более выразительным и типобезопасным, позволяя описывать сложные условия прямо в объявлениях.

### Зачем это знать iOS-разработчику?
1.  **Generic constraints:** Создание гибких, но типобезопасных обобщённых функций и типов.
2.  **Pattern matching:** Компактная фильтрация в циклах и `switch`.
3.  **Расширения протоколов:** Ограничение расширений конкретными типами.
4.  **Обработка ошибок:** Точная фильтрация по типам ошибок.
5.  **Читаемость:** Условия, интегрированные в синтаксис, часто читаются лучше.

---

## 1. `where` в дженериках ([[Generic]] Constraints)

### 1.1 Ограничения для типов

```swift
// Функция работает только с Equatable типами
func findIndex<T: Equatable>(of value: T, in array: [T]) -> Int? {
    for (index, item) in array.enumerated() {
        if item == value {
            return index
        }
    }
    return nil
}

// То же самое с where
func findIndexWhere<T>(of value: T, in array: [T]) -> Int? where T: Equatable {
    for (index, item) in array.enumerated() {
        if item == value {
            return index
        }
    }
    return nil
}
```

### 1.2 Ограничения для ассоциированных типов

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

// Расширение только для контейнеров с Equatable элементами
extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        for i in 0..<count {
            if self[i] == item {
                return true
            }
        }
        return false
    }
}
```

### 1.3 Множественные ограничения

```swift
// Несколько требований через запятую
func process<T>(_ value: T) where T: Equatable, T: Hashable {
    print("Value is equatable and hashable")
}

// Ограничение для двух дженериков
func compare<T, U>(_ a: T, _ b: U) -> Bool where T: Equatable, T == U {
    return a == b
}

// Ограничение на тип элемента коллекции
func sum<T: Sequence>(_ sequence: T) -> Int where T.Element == Int {
    return sequence.reduce(0, +)
}
```

### 1.4 Ограничения с классами

```swift
class Animal { }
class Dog: Animal { }
class Cat: Animal { }

// Функция работает только с Dog или его подклассами
func walk<T: Animal>(_ animal: T) where T: Dog {
    print("Walking the dog")
}

// walk(Dog())   // OK
// walk(Cat())   // Ошибка: Cat не является Dog
```

---

## 2. `where` в расширениях протоколов

```swift
protocol Stack {
    associatedtype Element
    mutating func push(_ element: Element)
    mutating func pop() -> Element?
    var isEmpty: Bool { get }
}

// Расширение для всех стеков
extension Stack {
    var top: Element? {
        var copy = self
        return copy.pop()
    }
}

// Расширение только для стеков с Equatable элементами
extension Stack where Element: Equatable {
    func contains(_ element: Element) -> Bool {
        var copy = self
        while let item = copy.pop() {
            if item == element {
                return true
            }
        }
        return false
    }
}

// Расширение для стеков, где Element — String
extension Stack where Element == String {
    func joined(separator: String = "") -> String {
        var result = ""
        var copy = self
        while let item = copy.pop() {
            result = item + (result.isEmpty ? "" : separator) + result
        }
        return result
    }
}
```

---

## 3. `where` в сопоставлении с образцом (Pattern Matching)

### 3.1 `switch` с `where`

```swift
let point = (x: 3, y: 4)

switch point {
case let (x, y) where x == y:
    print("На диагонали")
case let (x, y) where x < 0 && y < 0:
    print("III четверть")
case let (x, y) where x < 0:
    print("II четверть")
case let (x, y) where y < 0:
    print("IV четверть")
case let (x, y) where x == 0 || y == 0:
    print("На оси")
default:
    print("I четверть")
}

// Работа с опционалами
let optionalValue: Int? = 42

switch optionalValue {
case let value? where value > 50:
    print("Больше 50")
case let value? where value > 0:
    print("Положительное число")
case nil:
    print("nil")
default:
    print("Ноль или отрицательное")
}
```

### 3.2 `if case` и `guard case` с `where`

```swift
let result: Result<Int, Error> = .success(42)

if case .success(let value) = result, value > 0 {
    print("Успех с положительным значением: \(value)")
}

guard case .success(let value) = result, value != 0 else {
    print("Ошибка или ноль")
    return
}
print("Успех: \(value)")
```

### 3.3 `for-in` с `where`

```swift
let numbers = 1...10

// Только чётные числа
for number in numbers where number % 2 == 0 {
    print(number)  // 2, 4, 6, 8, 10
}

// С извлечением из опционалов
let optionalNumbers: [Int?] = [1, nil, 2, nil, 3]
for case let number? in optionalNumbers where number > 1 {
    print(number)  // 2, 3
}

// С индексами
let array = ["a", "b", "c", "d"]
for (index, value) in array.enumerated() where index % 2 == 0 {
    print("\(index): \(value)")  // 0: a, 2: c
}
```

---

## 4. `where` в `do-catch`

```swift
enum NetworkError: Error {
    case badURL
    case timeout
    case serverError(code: Int)
}

func fetchData() throws {
    throw NetworkError.serverError(code: 404)
}

do {
    try fetchData()
} catch NetworkError.serverError(let code) where code == 404 {
    print("Страница не найдена")
} catch NetworkError.serverError(let code) where code >= 500 {
    print("Ошибка сервера: \(code)")
} catch NetworkError.badURL {
    print("Неверный URL")
} catch {
    print("Другая ошибка: \(error)")
}
```

---

## 5. `where` в `case let` и `if var`

```swift
enum Expression {
    case number(Int)
    case addition(Expression, Expression)
    case multiplication(Expression, Expression)
}

let expr = Expression.addition(.number(5), .number(3))

// Рекурсивное вычисление с where
func evaluate(_ expr: Expression) -> Int {
    switch expr {
    case .number(let value):
        return value
    case .addition(let left, let right) where evaluate(left) == 0:
        return evaluate(right)
    case .addition(let left, let right) where evaluate(right) == 0:
        return evaluate(left)
    case .addition(let left, let right):
        return evaluate(left) + evaluate(right)
    case .multiplication(let left, let right):
        return evaluate(left) * evaluate(right)
    }
}
```

---

## 6. `where` в ассоциированных типах протоколов

```swift
protocol Iterator {
    associatedtype Element
    mutating func next() -> Element?
}

protocol Sequence {
    associatedtype Iterator: IteratorProtocol
    associatedtype Element where Element == Iterator.Element
    
    func makeIterator() -> Iterator
}

// Протокол с where для ассоциированного типа
protocol Collection: Sequence {
    associatedtype Index: Comparable
    associatedtype Element where Element == Iterator.Element
    
    subscript(position: Index) -> Element { get }
    var startIndex: Index { get }
    var endIndex: Index { get }
}
```

---

## 7. `where` в дженериках с несколькими параметрами

```swift
// Ограничение для двух типов
func zipEquatable<T, U>(_ a: T, _ b: U) -> Bool where T: Equatable, U: Equatable, T == U {
    return a == b
}

// Ограничение на соответствие протоколу
func compareHashable<T, U>(_ a: T, _ b: U) -> Bool where T: Hashable, U: Hashable, T == U {
    return a.hashValue == b.hashValue
}

// Ограничение на тип контейнера
func processContainer<C: Container>(_ container: C) where C.Item: CustomStringConvertible {
    for i in 0..<container.count {
        print(container[i].description)
    }
}
```

---

## 8. `where` с `available` (условная компиляция)

```swift
// Доступно только на iOS 15+
if #available(iOS 15.0, *) {
    // код для iOS 15+
}

// Более сложное условие
if #available(iOS 15.0, macOS 12.0, *) {
    // код для iOS 15+ или macOS 12+
}

// Для объявлений
@available(iOS 15.0, macOS 12.0, *)
func newFeature() { }
```

---

### Лучшие практики

#### 1. **Предпочитайте where в дженериках для сложных условий**

```swift
// ✅ Хорошо — читаемо
func process<T>(_ value: T) where T: Equatable, T: Hashable {
    // ...
}

// ❌ Плохо — перегружено
func process<T: Equatable & Hashable>(_ value: T) {
    // ...
}
```

#### 2. **Используйте where в циклах для фильтрации**

```swift
let numbers = 1...10

// ✅ Хорошо
for n in numbers where n % 2 == 0 {
    print(n)
}

// ❌ Плохо (лишняя вложенность)
for n in numbers {
    if n % 2 == 0 {
        print(n)
    }
}
```

#### 3. **Избегайте сложных where-условий**

```swift
// ❌ Плохо — слишком сложно
for item in items where item.isValid && item.count > 0 && !item.isDeleted && item.owner == currentUser {
    // ...
}

// ✅ Хорошо — вынести в вычисляемое свойство
let filteredItems = items.filter { $0.isValid && $0.count > 0 && !$0.isDeleted && $0.owner == currentUser }
for item in filteredItems {
    // ...
}
```

#### 4. **Используйте where в расширениях протоколов для type-safe [[API]]**

```swift
extension Array where Element: Numeric {
    var sum: Element {
        return reduce(0, +)
    }
}

let ints = [1, 2, 3]
print(ints.sum)  // 6

let doubles = [1.5, 2.5]
print(doubles.sum)  // 4.0
```

---

### Короткое правило

> **`where`** — добавляет условия и ограничения в дженериках, циклах, switch и do-catch.  
> Используйте для сужения области действия generic-кода и фильтрации в паттерн-матчинге.  
> Не усложняйте — выносите сложные условия в вычисляемые свойства.

---

### Итог

**`where`** в Swift:

1.  **Generic constraints:** Ограничения для типов в дженериках
2.  **Расширения протоколов:** Условное добавление функциональности
3.  **Pattern matching:** Фильтрация в `switch`, `for-in`, `if case`
4.  **Do-catch:** Фильтрация ошибок
5.  **Ассоциированные типы:** Ограничения в протоколах

`where` делает код более выразительным и типобезопасным, позволяя описывать сложные условия прямо в синтаксисе объявлений.