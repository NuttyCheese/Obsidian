`reduce` — один из самых мощных и универсальных методов в стандартной библиотеке Swift. Он позволяет **свести** (reduce / fold) всю коллекцию к **одному значению** путём последовательного применения операции.

Существует **две основные версии**:

- `reduce(_:_:)` — классическая, иммутабельная
- `reduce(into:_:)` — мутабельная, более эффективная для сложных типов

## 1. Классический reduce

```swift
func reduce<Result>(
    _ initialResult: Result,
    _ nextPartialResult: (Result, Element) throws -> Result
) rethrows -> Result
```

- аккумулятор передаётся **по значению**
- на каждом шаге создаётся **новое** значение аккумулятора
- идеален для **неизменяемых** операций (числа, строки в небольших объёмах, [[enum]], [[struct]])

### Примеры

```swift
let numbers = [1, 2, 3, 4, 5]

// Сумма
let sum = numbers.reduce(0, +)                      // 15
let sumLong = numbers.reduce(0) { $0 + $1 }

// Произведение
let product = numbers.reduce(1, *)                  // 120

// Максимум
let maxValue = numbers.reduce(Int.min, max)         // 5

// Конкатенация строк (маленькие объёмы — ок)
let sentence = ["Swift", "is", "awesome"].reduce("") { $0 + " " + $1 }.trimmingCharacters(in: .whitespaces)
// → "Swift is awesome"
```

## 2. reduce(into:) — мутабельная и эффективная версия

```swift
func reduce<Result>(
    into initialResult: Result,
    _ updateAccumulatingResult: (inout Result, Element) throws -> Void
) rethrows -> Result
```

- аккумулятор передаётся **по ссылке (inout)**
- вы **изменяете** один и тот же объект на месте
- **O(n)** по памяти и времени для мутабельных типов ([[Array]], [[Dictionary]], [[String]], [[Set]] и т.д.)

### Почему это критически важно

```swift
// ПЛОХО — O(n²) по времени и памяти
let appended = (1...10000).reduce([]) { $0 + [$1] }

// ХОРОШО — O(n)
let appendedEfficient = (1...10000).reduce(into: []) { $0.append($1) }
```

То же самое касается словарей, множеств, строк и любых типов, где операция `+` / `+=` создаёт новый экземпляр.

### Популярные примеры reduce(into:)

```swift
// Группировка по ключу (самый частый кейс)
let transactions = [
    ("Food", 1200), ("Transport", 800), ("Food", 450), ("Salary", 150000)
]

let byCategory = transactions.reduce(into: [String: [Int]]()) { result, transaction in
    let (category, amount) = transaction
    result[category, default: []].append(amount)
}

// Подсчёт частоты элементов
let words = ["cat", "dog", "cat", "bird", "cat", "dog"]
let frequency = words.reduce(into: [String: Int]()) { $0[$1, default: 0] += 1 }
// → ["cat": 3, "dog": 2, "bird": 1]

// Уникальные элементы (аналог Set)
let unique = numbers.reduce(into: Set<Int>()) { $0.insert($1) }

// Построение строки эффективно
let csv = rows.reduce(into: "") { result, row in
    result += row.joined(separator: ",") + "\n"
}
```

## Сравнение reduce vs reduce(into:)

| Критерий                        | reduce(_:_:)                          | reduce(into:_:)                        | Когда выбрать |
|---------------------------------|----------------------------------------|----------------------------------------|---------------|
| Аккумулятор                     | По значению (копируется)               | По ссылке (inout)                      | — |
| Подходит для                    | Числа, Bool, enum, маленькие строки    | Array, Dictionary, Set, String, struct | — |
| Производительность (Array/Dict) | O(n²) в худшем случае                  | O(n)                                   | reduce(into:) |
| Иммутабельный стиль             | Да                                     | Нет (мутирует аккумулятор)             | reduce |
| Читаемость для простых случаев  | Выше                                   | Чуть ниже                              | reduce |
| Группировка, сборка коллекций   | Неэффективно / невозможно              | Идеально                               | reduce(into:) |

**Правило 2026 года**  
Если аккумулятор — **коллекция** (массив, словарь, множество, строка Builder и т.п.) → **всегда** используйте `reduce(into:)`  
Если аккумулятор — примитив ([[Int]], [[Double]], [[Bool]], [[enum]]) → можно использовать обычный `reduce`

## Комбинации с [[lazy]] и другими методами

```swift
// Один проход — фильтр + преобразование + свёртка
let totalEvenDoubled = numbers
    .lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .reduce(0, +)
```

## reduce как фундамент других операций

```swift
// Самостоятельная реализация map через reduce(into:)
extension Sequence {
    func myMap<T>(_ transform: (Element) -> T) -> [T] {
        reduce(into: []) { $0.append(transform($1)) }
    }
}

// Самостоятельная реализация filter
extension Sequence {
    func myFilter(_ predicate: (Element) -> Bool) -> [Element] {
        reduce(into: []) { if predicate($1) { $0.append($1) } }
    }
}
```

## Продвинутые примеры 2025–2026

```swift
// Накопительная сумма (cumulative sum)
let cumulative = numbers.reduce(into: (sum: 0, result: [Int]())) { state, num in
    state.sum += num
    state.result.append(state.sum)
}.result
// → [1, 3, 6, 10, 15]

// Построение JSON-подобной структуры
struct CategorySummary {
    var total: Decimal = 0
    var count: Int = 0
    var names: [String] = []
}

let summary = items.reduce(into: [String: CategorySummary]()) { dict, item in
    dict[item.category, default: .init()].total += item.price
    dict[item.category]!.count += 1
    dict[item.category]!.names.append(item.name)
}
```

## Итог — современные рекомендации

- Сумма, произведение, min/max, логические операции → `reduce`
- Построение массива, словаря, множества, строки, любой мутабельной структуры → `reduce(into:)`
- Группировка, частотный анализ, агрегация по ключу → **только** `reduce(into:)`
- Критичная производительность с большими данными → `reduce(into:)` + `lazy`
- Хотите иммутабельный, чисто-функциональный стиль → `reduce` (для примитивов)

`reduce` и особенно `reduce(into:)` — это **самый универсальный инструмент** для агрегации в Swift. Многие сложные операции (группировка, уникализация, построение отчётов) проще и эффективнее всего выражаются именно через них.
