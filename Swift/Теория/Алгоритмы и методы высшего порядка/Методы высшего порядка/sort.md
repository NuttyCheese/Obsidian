#swift #standard_library #collection #sequence #sorting #algorithm #comparable #order
# sort() и sorted() в Swift

Методы `sort()` и `sorted()` — это два самых часто используемых инструмента для сортировки коллекций в [[Swift]].

Главное различие между ними:

| Метод       | Что делает                              | Изменяет исходную коллекцию? | Возвращает          | Когда использовать                          |
|-------------|-----------------------------------------|------------------------------|---------------------|---------------------------------------------|
| `sort()`    | Сортирует **на месте** (in-place)       | **Да**                       | `Void`              | Когда можно и нужно изменить исходный массив |
| `sorted()`  | Возвращает **новый** отсортированный массив | **Нет**                      | `[Element]`         | Когда нужно сохранить исходный порядок      |

Оба метода доступны на типах, реализующих протокол `MutableCollection` (для `sort()`) и `Sequence` (для `sorted()`).

## 1. Базовая сортировка (по умолчанию)

```swift
var numbers = [5, 2, 7, 3, 1, 8, 4]
numbers.sort()
print(numbers)          // [1, 2, 3, 4, 5, 7, 8]

// или immutable вариант
let sortedNumbers = [5, 2, 7, 3, 1, 8, 4].sorted()
```

По умолчанию используется оператор `<` (требуется [[Comparable]]).

## 2. Сортировка в обратном порядке

```swift
var scores = [85, 92, 78, 100, 65]
scores.sort(by: >)              // по убыванию
// или
scores.sort { $0 > $1 }

// sorted-вариант
let descending = scores.sorted(by: >)
```

## 3. Сортировка с замыканием (самый гибкий вариант)

```swift
var people = [
    ("Bob",   25),
    ("Alice", 30),
    ("Charlie", 35),
    ("David", 28)
]

// По возрасту (возрастание)
people.sort { $0.1 < $1.1 }

// По имени (лексикографически)
people.sort { $0.0 < $1.0 }

// По нескольким критериям (приоритет возраста, затем имени)
people.sort {
    if $0.1 != $1.1 { return $0.1 < $1.1 }
    return $0.0 < $1.0
}
```

## 4. Сортировка структур и классов (самый частый сценарий)

```swift
struct User: Identifiable {
    let id: UUID
    let name: String
    let age: Int
    let score: Double
    let isActive: Bool
}

var users: [User] = // ...

// Вариант 1 — через замыкание
users.sort { $0.age < $1.age }

// Вариант 2 — через KeyPath + сравнение (Swift 5.3+)
users.sort { $0.age < $1.age }
users.sort { $0.score > $1.score }           // по убыванию

// Вариант 3 — несколько критериев (очень читаемо)
users.sort {
    ($0.isActive ? 1 : 0, $0.score, $0.name) <
    ($1.isActive ? 1 : 0, $1.score, $1.name)
}

// Вариант 4 — самый современный (Swift 5.7+ с partial key paths)
users.sort {
    ($0[keyPath: \.isActive], $0.score, $0.name) >
    ($1[keyPath: \.isActive], $1.score, $1.name)
}
```

## 5. Сортировка по нескольким полям (реальные примеры)

```swift
// Сортировка задач: сначала по приоритету (высокий → низкий), потом по дедлайну
tasks.sort {
    if $0.priority != $1.priority {
        return $0.priority > $1.priority   // высокий приоритет первым
    }
    return $0.dueDate < $1.dueDate         // ранний дедлайн первым
}

// Сортировка транзакций: сначала доходы, потом расходы, внутри — по дате
transactions.sort {
    let lhsIsIncome = lhs.amount > 0
    let rhsIsIncome = rhs.amount > 0
    
    if lhsIsIncome != rhsIsIncome {
        return lhsIsIncome ? false : true   // доходы раньше расходов
    }
    
    return lhs.date < rhs.date
}
```

## 6. sorted(by:) vs sort(by:) — производительность

| Коллекция       | `sort()` (in-place)  | `sorted()` (копия) | Разница в памяти | Когда критично  |
| --------------- | -------------------- | ------------------ | ---------------- | --------------- |
| [[Array]]       | O(n log n), in-place | O(n log n) + O(n)  | +O(n)            | Большие массивы |
| ContiguousArray | То же                | То же              | +O(n)            | —               |
| ArraySlice      | Не поддерживается    | Да                 | —                | Только sorted   |

**Рекомендация 2026 года**  
- Если массив большой (> 10 000 элементов) и вы больше не используете исходный порядок → `sort()`  
- Если исходный массив нужен дальше → `sorted()`  
- В UI-коде (таблицы, коллекции) почти всегда `sorted()` — безопаснее

## 7. Полезные расширения и паттерны

```swift
extension Sequence {
    func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        sorted { $0[keyPath: keyPath] < $1[keyPath: keyPath] }
    }
    
    func sortedDescending<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        sorted { $0[keyPath: keyPath] > $1[keyPath: keyPath] }
    }
}

// Использование
let sortedByScore = users.sorted(by: \.score)
let topByName     = users.sortedDescending(by: \.name)
```

## 8. Часто задаваемые вопросы и ловушки

- `sort()` на `let`-массиве? → **Ошибка компиляции**  
  → используйте `sorted()`

- Сортировка опционалов?  
  ```swift
  let optionalAges: [Int?] = [nil, 25, 30, nil, 18]
  let sorted = optionalAges.sorted { ($0 ?? Int.max) < ($1 ?? Int.max) }
  ```

- Сортировка по enum с rawValue  
  ```swift
  enum Priority: Int, CaseIterable { case low = 0, medium = 1, high = 2 }
  tasks.sort { $0.priority.rawValue > $1.priority.rawValue }
  ```

- Стабильна ли сортировка?  
  Да — и `sort()`, и `sorted()` используют **стабильный** алгоритм (Timsort-подобный) с Swift 5+

## Итог — современные рекомендации

| Ситуация                              | Рекомендация              |
|---------------------------------------|---------------------------|
| Нужно изменить исходный массив        | `sort()`                  |
| Исходный массив нужен дальше          | `sorted()`                |
| Сортировка по одному полю             | `sorted(by: \.field)`     |
| Сортировка по нескольким критериям    | замыкание с if / кортеж   |
| Большие массивы (>50k элементов)      | `sort()` предпочтительнее |
| UI / список / таблица                 | почти всегда `sorted()`   |

`sort()` и `sorted()` — это те методы, которые вы используете каждый день.  
Главное — помнить про **in-place vs копия** и про **KeyPath**-синтаксис — он делает код намного чище и выразительнее.
