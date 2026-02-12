#algoritms #higher-order_func #Swift 
# Функциональное программирование в Swift: Основные функции высшего порядка

Функции высшего порядка (Higher-Order Functions) — это функции, которые принимают другие функции (замыкания) в качестве аргументов и/или возвращают функции как результат. В [[Swift]] они являются фундаментом функционального стиля программирования и используются практически в каждом реальном проекте.

Ниже приведён обзор самых важных и часто используемых функций высшего порядка из стандартной библиотеки Swift.

## 1. [[map]]
Преобразует каждый элемент коллекции по заданному правилу.  
Результат — **новая коллекция** той же длины.

```swift
let numbers = [1, 2, 3, 4, 5]
let squared = numbers.map { $0 * $0 }               // [1, 4, 9, 16, 25]
let names = ["anna", "ben", "clara"]
let capitalized = names.map { $0.capitalized }      // ["Anna", "Ben", "Clara"]
```

Современный стиль с KeyPath (Swift 5.2+):

```swift
struct User { let name: String; let age: Int }
let users: [User] = …
let names = users.map(\.name)                       // [String]
let ages  = users.map(\.age)                        // [Int]
```

## 2. [[compactMap]]
Преобразует элементы и **автоматически убирает nil**-результаты.

```swift
let strings = ["12", "45", "seven", "19", "0", "abc"]
let numbers = strings.compactMap { Int($0) }        // [12, 45, 19, 0]
```

Классический пример — безопасное извлечение значений:

```swift
let optionalUsers: [User?] = [user1, nil, user3, nil]
let validUsers = optionalUsers.compactMap { $0 }    // [User]
```

## 3. [[flatMap]]
Расплющивает (flatten) вложенные коллекции после применения преобразования.

```swift
let nested = [[1,2], [3,4,5], [], [6]]
let flat = nested.flatMap { $0 }                    // [1,2,3,4,5,6]

let departments = [
    ["iOS": ["Anna", "Ben"]],
    ["Backend": ["Clara", "David"]]
]
let allNames = departments.flatMap { $0.values }    // [["Anna","Ben"], ["Clara","David"]]
    .flatMap { $0 }                                 // ["Anna","Ben","Clara","David"]
```

**Важно 2026 года**:  
`flatMap { Int($0) }` → **устарело** → используйте `compactMap { Int($0) }`

## 4. [[filter]]
Оставляет только элементы, удовлетворяющие условию.

```swift
let numbers = 1...20
let evens = numbers.filter { $0 % 2 == 0 }          // [2,4,6,...,20]

let activeUsers = users.filter(\.isActive)
let expensive = products.filter { $0.price > 1000 }
```

## 5. [[reduce]]
Сворачивает (агрегирует) коллекцию в одно значение.

```swift
let sum = numbers.reduce(0, +)                     // 15 (короткая запись)
let sumLong = numbers.reduce(0) { $0 + $1 }         // то же самое

let longest = words.reduce("") { current, next in
    next.count > current.count ? next : current
}
```

С мутабельным аккумулятором (часто быстрее):

```swift
let concatenated = words.reduce(into: "") { $0 += $1 + " " }
```

## 6. [[forEach]]
Выполняет действие для каждого элемента (только сайд-эффекты).

```swift
users.forEach { user in
    Analytics.track("UserViewed", ["id": user.id])
}

[1,2,3,4,5].forEach { print($0) }
```

**Не используйте** для трансформации — это антипаттерн.

## 7. [[contains]] / contains(where:)
Проверяет наличие элемента или выполнение условия.

```swift
numbers.contains(42)                        // Bool
users.contains { $0.age < 18 }              // есть ли несовершеннолетний?
users.contains(where: \.isBlocked)          // есть ли заблокированный?
```

## 8. sorted ([[sort]])
Возвращает **новую** отсортированную коллекцию.

```swift
let sortedAsc  = numbers.sorted()                   // по возрастанию
let sortedDesc = numbers.sorted(by: >)              // по убыванию

users.sorted { $0.age < $1.age }                    // по возрасту
users.sorted { $0.name < $1.name }                  // по имени

// По нескольким критериям
users.sorted {
    if $0.priority != $1.priority { return $0.priority > $1.priority }
    return $0.name < $1.name
}
```

## 9. [[split]] (для строк)
Разбивает строку на подстроки.

```swift
let csv = "name,age,city"
let fields = csv.split(separator: ",")              // ["name", "age", "city"]

let sentence = "Hello   world   Swift"
let words = sentence.split { $0.isWhitespace }      // без пустых строк
```

## 10. Дополнительные полезные функции высшего порядка (часто используются вместе)

- `first(where:)`
- `firstIndex(where:)`
- `last(where:)`
- `min(by:) / max(by:)`
- `allSatisfy(_:)`
- `dropFirst(_:) / dropLast(_:)`
- `prefix(_:) / suffix(_:)`
- `enumerated()`
- `joined() / joined(separator:)`

## Типичный функциональный пайплайн (2026)

```swift
let result = users
    .filter { $0.isActive && $0.age >= 18 }
    .sorted { $0.lastActive > $1.lastActive }
    .prefix(10)
    .map { user in
        "\(user.name) (\(user.age))"
    }
    .joined(separator: "\n")

print(result)
```

## Когда какую функцию выбрать (шпаргалка)

| Задача                                            | Функция(и)                     |
| ------------------------------------------------- | ------------------------------ |
| Преобразовать каждый элемент                      | `map`                          |
| Преобразовать и убрать [[nil]]                    | `compactMap`                   |
| Расплющить вложенные коллекции                    | `flatMap`                      |
| Оставить только подходящие элементы               | `filter`                       |
| Свернуть в одно значение                          | `reduce`, `reduce(into:)`      |
| Выполнить сайд-эффекты                            | `forEach`                      |
| Проверить наличие / условие                       | `contains`, `contains(where:)` |
| Отсортировать                                     | `sorted`, `sorted(by:)`        |
| Найти первый подходящий элемент                   | `first(where:)`                |
| Найти индекс первого подходящего                  | `firstIndex(where:)`           |
| Проверить, что все элементы удовлетворяют условию | `allSatisfy`                   |

Эти методы — основа **функционального стиля** в Swift. Они делают код **короче**, **безопаснее** (меньше мутаций), **выразительнее** и легче тестируемым.
