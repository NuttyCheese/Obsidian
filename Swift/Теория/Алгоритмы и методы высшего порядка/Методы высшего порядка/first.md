#algoritms #swift #higher-order_func
# first и first(where:) в Swift

В [[Swift]] методы семейства `first` — это одни из самых часто используемых инструментов для поиска **первого подходящего элемента** в коллекции или последовательности. Они чрезвычайно важны, потому что:

- останавливают поиск **как только нашли первый подходящий элемент** (early return)
- возвращают **опционал** (`Element?`)
- работают на всех типах, реализующих `Sequence`

## Основные варианты (2025–2026)

```swift
// 1. Самый популярный и полезный
func first(where predicate: (Element) throws -> Bool) rethrows -> Element?

// 2. Просто первый элемент (если коллекция не пустая)
var first: Element? { get }    // свойство, доступно на Collection

// 3. Первый элемент, приводящий к успешному преобразованию
func first<T>(where transform: (Element) throws -> T?) rethrows -> T?

// 4. Специфично для Range, ClosedRange и т.п.
extension RangeExpression {
    var first: Bound? { ... }
}
```

## Ключевые отличия от похожих методов

| Метод                     | Останавливает при первом совпадении? | Возвращает          | Когда использовать                              |
|---------------------------|---------------------------------------|----------------------|--------------------------------------------------|
| `first(where:)`           | Да                                    | `Element?`           | Нужно найти **первый** элемент по условию       |
| `filter(...).first`       | Нет (проходит всю коллекцию)          | `Element?`           | Никогда — это антипаттерн                       |
| `first` (свойство)        | —                                     | `Element?`           | Просто нужен любой первый элемент               |
| `firstIndex(where:)`      | Да                                    | `Index?`             | Нужен индекс, а не сам элемент                  |
| `contains(where:)`        | Да                                    | `Bool`               | Нужно только проверить наличие                  |

## Примеры использования

### 1. Классический поиск первого подходящего

```swift
let scores = [78, 92, 65, 88, 45, 99, 71]

if let firstExcellent = scores.first(where: { $0 >= 90 }) {
    print("Первый отличный результат: \(firstExcellent)")   // 92
}
```

### 2. Поиск по ключевому полю (самый частый паттерн)

```swift
struct Order {
    let id: String
    let status: OrderStatus
    let createdAt: Date
}

enum OrderStatus: String { case pending, processing, shipped, delivered, cancelled }

let orders: [Order] = // ...

// Первый невыполненный заказ
let firstPending = orders.first { $0.status == .pending }

// Первый самый старый заказ
let oldestOrder = orders.first { _ in true }   // или просто orders.first

// Первый заказ со статусом shipped за последние 7 дней
let recentShipped = orders.first {
    $0.status == .shipped &&
    $0.createdAt > Date().addingTimeInterval(-7*24*3600)
}
```

### 3. first + KeyPath (очень читаемый стиль с Swift 5.2+)

```swift
let users = [
    User(name: "Anna", age: 17, isActive: true),
    User(name: "Ben",  age: 24, isActive: false),
    User(name: "Clara", age: 19, isActive: true)
]

// Первый активный пользователь
let firstActive = users.first(where: \.isActive)

// Первый совершеннолетний
let firstAdult = users.first { $0.age >= 18 }
let firstAdultKP = users.first { $0[keyPath: \.age] >= 18 }
```

### 4. first(where:) vs [[filter]].first (очень важное сравнение)

```swift
// ПЛОХО — ненужный проход по всей коллекции + аллокация массива
let wrong = transactions
    .filter { $0.amount > 1000 && $0.isInternational }
    .first

// ХОРОШО — останавливается на первом совпадении
let correct = transactions.first {
    $0.amount > 1000 && $0.isInternational
}
```

### 5. first с преобразованием (очень мощный паттерн)

```swift
let strings = ["12", "abc", "45", "seven", "19"]

// Первый успешно распарсенный Int
if let firstNumber = strings.first(where: { Int($0) }) {
    print("Первое число: \(firstNumber)")   // 12
}

// Более явный вариант
let firstParsed = strings.first { str -> Int? in
    Int(str)
}
```

### 6. first в словарях и множествах

```swift
let settings: [String: String] = [
    "theme": "dark",
    "language": "ru",
    "currency": "RUB"
]

// Первый ключ, начинающийся на "c"
let firstCKey = settings.first { $0.key.hasPrefix("c") }?.key   // "currency"

// Самое маленькое значение (если порядок не важен)
let smallestValue = settings.values.min()
```

### 7. first в [[String]]

```swift
let text = "Привет, мир! Hello, world! こんにちは"

// Первый символ, являющийся цифрой
let firstDigit = text.first { $0.isNumber }

// Первый не-ASCII символ
let firstNonASCII = text.first { !$0.isASCII }
```

### 8. Реальные сценарии 2025–2026

```swift
// Первый валидный токен авторизации
let validToken = tokens.first { $0.isValid && !$0.isExpired }

// Первый элемент с максимальным приоритетом
let topPriorityTask = tasks.first {
    $0.priority == tasks.map(\.priority).max()
}

// Первый подходящий координат в маршруте
let firstWaypointInRegion = route.waypoints.first {
    region.contains($0.coordinate)
}
```

## Производительность и лучшие практики

- `first(where:)` — **O(n)** в худшем случае, но часто **очень рано завершается**
- `first` (свойство) — **O(1)** на большинстве коллекций
- Никогда не пишите `filter {}.first` — это всегда хуже по производительности и памяти
- Если вам нужен индекс — используйте `firstIndex(where:)`
- Для ленивых последовательностей `first(where:)` остаётся ленивым

```swift
// Рекомендуемый паттерн поиска с ранним выходом
guard let user = users.first(where: { $0.id == targetID }) else {
    throw UserError.notFound
}
```

## Итог — когда использовать что

- Хотите **первый элемент вообще** → `.first`
- Ищете **первый по условию** → `.first(where:)`
- Нужен **индекс** → `.firstIndex(where:)`
- Нужно **все подходящие** → `filter`
- Нужно **проверить наличие** → `contains(where:)`
- Хотите **первый после преобразования** → `.first { transform($0) != nil }`

Методы `first` и `first(where:)` — это маленькие, но невероятно мощные инструменты, которые делают код Swift безопасным, читаемым и эффективным. Используйте их вместо `filter(...).first` — и ваш код станет заметно лучше.
