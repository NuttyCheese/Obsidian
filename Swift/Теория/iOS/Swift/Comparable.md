**`Comparable`** — это протокол в [[Swift]], который позволяет сравнивать экземпляры типа между собой с помощью операторов `<`, `<=`, `>`, `>=`.

- Требует реализации **единственного метода** `compare(to:)`, который возвращает `ComparisonResult`
- Автоматически даёт возможность использовать `<`, `>`, `<=`, `>=`, `min(_:_:)`, `max(_:_:)`, сортировку коллекций и т.д.
- Очень часто используется вместе с `Equatable` (для `==` / `!=`)

> Проще говоря: `Comparable` = «мой тип можно сравнивать с помощью < > ≤ ≥»

### 1. Основные требования протокола (2026 актуально)

```swift
public protocol Comparable : Equatable {
    func compare(to other: Self) -> ComparisonResult
}
```

- `ComparisonResult` — это `enum` с тремя значениями:
  - `.orderedAscending`   → self < other
  - `.orderedSame`        → self == other
  - `.orderedDescending`  → self > other

**Важно**:  
`Comparable` **наследует** [[Equatable]], поэтому тип, conforming к `Comparable`, **обязательно** должен реализовать `==`.

### 2. Самый простой и рекомендуемый способ реализации (2026 стандарт)

```swift
struct Point: Comparable {
    let x: Double
    let y: Double
    
    // Сравниваем по расстоянию от начала координат
    static func < (lhs: Point, rhs: Point) -> Bool {
        let distL = lhs.x * lhs.x + lhs.y * lhs.y
        let distR = rhs.x * rhs.x + rhs.y * rhs.y
        return distL < distR
    }
    
    // Эквивалентно (требуется Equatable)
    static func == (lhs: Point, rhs: Point) -> Bool {
        lhs.x == rhs.x && lhs.y == rhs.y
    }
}

// Использование
let p1 = Point(x: 3, y: 4)     // расстояние 5
let p2 = Point(x: 1, y: 1)     // расстояние ≈1.41
print(p1 > p2)                 // true
print(min(p1, p2))             // Point(x: 1, y: 1)
```

**Альтернативный (ещё более популярный) способ** — через реализацию `compare(to:)`:

```swift
extension Point: Comparable {
    func compare(to other: Point) -> ComparisonResult {
        let distSelf = x * x + y * y
        let distOther = other.x * other.x + other.y * other.y
        
        if distSelf < distOther { return .orderedAscending }
        if distSelf > distOther { return .orderedDescending }
        return .orderedSame
    }
}
```

Оба способа работают одинаково — Swift сам генерирует операторы из `compare(to:)` или наоборот.

### 3. Самые частые паттерны Comparable в 2026 году

#### Паттерн 1: Сортировка по нескольким критериям

```swift
struct Task: Comparable {
    let priority: Int
    let createdAt: Date
    let title: String
    
    static func < (lhs: Task, rhs: Task) -> Bool {
        if lhs.priority != rhs.priority {
            return lhs.priority > rhs.priority  // более высокий приоритет — раньше
        }
        return lhs.createdAt < rhs.createdAt     // потом — по времени создания
    }
}

let tasks = [
    Task(priority: 1, createdAt: Date(), title: "Urgent"),
    Task(priority: 3, createdAt: Date().addingTimeInterval(-3600), title: "Old medium"),
    Task(priority: 1, createdAt: Date().addingTimeInterval(-7200), title: "Very old urgent")
]

let sorted = tasks.sorted()  // Автоматически сортирует по приоритету, затем по времени
```

#### Паттерн 2: Comparable + Equatable + [[Hashable]] (для [Set] / [[Dictionary]])

```swift
struct Score: Comparable, Hashable {
    let player: String
    let points: Int
    
    static func < (lhs: Score, rhs: Score) -> Bool {
        lhs.points > rhs.points  // больше очков — "меньше" (раньше в лидерборде)
    }
    
    static func == (lhs: Score, rhs: Score) -> Bool {
        lhs.player == rhs.player && lhs.points == rhs.points
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(player)
        hasher.combine(points)
    }
}
```

#### Паттерн 3: Comparable в [[Range]] / ClosedRange

```swift
let range = 1...10
print(5 < range.upperBound)   // true
print(range.contains(11))     // false
```

### 4. Лучшие практики Comparable в Swift 2026

- **Реализуйте `<` и `==`** — это самый читаемый и понятный способ  
- **Соблюдайте строгое отношение порядка** (трихотомия):  
  - a < b → !(b < a)  
  - !(a < b) && !(b < a) → a == b  
  - a < b && b < c → a < c  
- **Не нарушайте транзитивность** — иначе `sorted()` может крашнуться или дать неверный результат  
- **Для сортировки по убыванию** — меняйте знак сравнения (`lhs.points > rhs.points`)  
- **Swift 6 strict concurrency** — `Comparable` полностью `Sendable`, если `Self` — `Sendable`  
- **Документируйте** — пиши комментарий «Comparable — сортировка по приоритету, затем по времени создания»

**Короткий девиз 2026**:
> `Comparable` — это когда твой тип можно **сортировать**, **сравнивать** и использовать в `min`/`max` / `Range`.  
> В 2026 году реализуй **`<` и `==`**, соблюдай трихотомию и используй `sorted()`, `min()`, `max()` — это один из самых мощных инструментов стандартной библиотеки.
