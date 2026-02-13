#swift #dictionary #collections #higher_order_function #data_transformation
# Dictionary(grouping:by:) в Swift

`Dictionary(grouping:by:)` — это один из самых элегантных и часто используемых инициализаторов словаря в современном [[Swift]] (добавлен в Swift 4).

Он решает очень распространённую задачу: **разбить последовательность элементов на группы по какому-либо критерию** и получить результат в виде удобного словаря `[Key: [Element]]`.

## Сигнатура (2025–2026 годы)

```swift
init<S>(
    grouping values: S,
    by keyForValue: (S.Element) throws -> Key?
) rethrows
where S : Sequence, Key : Hashable
```

С Swift 5.1+ существует **вторая, более часто используемая перегрузка** с Key Path:

```swift
init<S>(
    grouping values: S,
    by keyPath: KeyPath<S.Element, Key>
) where S : Sequence, Key : Hashable
```

## Что он делает (шаг за шагом)

1. Создаёт пустой словарь `[Key: [Value]]`
2. Проходит по каждому элементу последовательности `values`
3. Для каждого элемента вычисляет ключ (либо через замыкание, либо через key path)
4. Если ключ уже существует → добавляет элемент в существующий массив
5. Если ключа ещё нет → создаёт новый массив с этим элементом
6. Возвращает готовый словарь

Важно:  
- Если замыкание возвращает [[nil]] → элемент **игнорируется** (не попадает ни в одну группу)  
- Порядок элементов внутри каждой группы **сохраняется** (соответствует порядку в исходной последовательности)

## Примеры

### 1. Базовый пример — чётные / нечётные

```swift
let numbers = Array(1...12)

let grouped = Dictionary(grouping: numbers) { n in
    n.isMultiple(of: 2) ? "even" : "odd"
}

// Результат (примерный порядок):
// [
//   "even": [2, 4, 6, 8, 10, 12],
//   "odd":  [1, 3, 5, 7, 9, 11]
// ]
```

### 2. Самый популярный паттерн — группировка по KeyPath

```swift
struct Transaction {
    let id: UUID
    let amount: Decimal
    let category: Category
    let date: Date
    let isExpense: Bool
}

enum Category: String, CaseIterable {
    case food, transport, entertainment, salary, investment
}

let transactions: [Transaction] = // ...

let byCategory = Dictionary(grouping: transactions, by: \.category)
// Тип: [Category: [Transaction]]

let expensesByCategory = Dictionary(grouping: transactions.filter(\.isExpense), by: \.category)
```

### 3. Группировка по первой букве имени (очень частый кейс в контактах, списке сотрудников)

```swift
struct Person {
    let fullName: String
    // ...
}

let people: [Person] = // ...

let byInitial = Dictionary(grouping: people) { person in
    String(person.fullName.prefix(1)).uppercased()
}

// или короче (Swift 5.7+ с partial key path)
let byInitialShort = Dictionary(grouping: people, by: \.fullName.first!.uppercased())
```

### 4. Группировка по диапазонам (возрастные группы)

```swift
struct User {
    let name: String
    let age: Int
}

let users: [User] = // ...

let ageGroups = Dictionary(grouping: users) { user -> String in
    switch user.age {
    case 0..<18:    "child"
    case 18..<30:   "young adult"
    case 30..<50:   "adult"
    case 50...:     "senior"
    default:        "unknown"
    }
}
```

### 5. Игнорирование элементов (ключ = nil)

```swift
let items = ["apple", "banana", "", "kiwi", " "]

let nonEmpty = Dictionary(grouping: items) { fruit -> String? in
    let trimmed = fruit.trimmingCharacters(in: .whitespaces)
    return trimmed.isEmpty ? nil : trimmed.first?.uppercased()
}

// В результате пустые строки и пробелы просто пропадают
```

### 6. Группировка по дате (день / неделя / месяц)

```swift
// Группировка по месяцу и году
let byMonth = Dictionary(grouping: transactions) { t in
    Calendar.current.dateComponents([.year, .month], from: t.date)
}

// Группировка по ISO неделе (очень полезно для отчётов)
let byWeek = Dictionary(grouping: transactions) { t in
    Calendar.current.dateComponents([.yearForWeekOfYear, .weekOfYear], from: t.date)
}
```

## Сравнение способов группировки (2025–2026)

| Способ                               | Читаемость | Производительность | Универсальность | Когда использовать                               |
| ------------------------------------ | ---------- | ------------------ | --------------- | ------------------------------------------------ |
| `Dictionary(grouping:by:)`           | ★★★★★      | ★★★★☆              | ★★★☆☆           | Нужен именно `[Key: [Element]]`                  |
| [[reduce]]`(into:)`                  | ★★★☆☆      | ★★★★★              | ★★★★★           | Нужно считать, суммировать, строить сложные типы |
| `grouped by key path + Swift 5.7+`   | ★★★★★      | ★★★★☆              | ★★★★☆           | Современный и самый чистый код                   |
| [[for-in]] + [[if let]] / [[switch]] | ★★☆☆☆      | ★★★★★              | ★★★★★           | Очень сложная логика или мутабельные структуры   |

## Типичные ошибки и антипаттерны

```swift
// ❌ Плохо: два прохода
let groups = Dictionary(grouping: items, by: \.category)
let counts = groups.mapValues { $0.count }

// Лучше — один проход (если нужна только статистика)
let counts = items.reduce(into: [Category: Int]()) { result, item in
    result[item.category, default: 0] += 1
}
```

```swift
// ❌ Не работает (компилятор не пропустит)
let wrong = Dictionary(grouping: users, by: \.age)          // age: Int — ок
let wrong2 = Dictionary(grouping: users, by: \.age / 10)    // ошибка — не Hashable
```

## Полезные комбинации (2025+)

```swift
// Сортировка групп и элементов внутри
let sortedGroups = Dictionary(grouping: transactions, by: \.category)
    .sorted { $0.key.rawValue < $1.key.rawValue }
    .map { ($0.key, $0.value.sorted { $0.date < $1.date }) }

// Самые большие группы
let topCategories = Dictionary(grouping: transactions, by: \.category)
    .mapValues { $0.count }
    .sorted { $0.value > $1.value }
    .prefix(3)
```

## Итог — когда использовать именно Dictionary(grouping:by:)

Используй **всегда**, когда тебе нужна структура:

- `[HashableKey: [Element]]`
- для секций таблицы / коллекции
- для отчётов по категориям / статусам / датам
- когда порядок элементов внутри группы важен
- когда хочешь максимально выразительный и безопасный код

Не используй, если:

- тебе нужны агрегации (сумма, среднее, количество, мин/макс) → `reduce(into:)`
- тебе нужна другая структура результата (не массив)
- производительность критична и ты можешь написать более оптимальный цикл

`Dictionary(grouping:by:)` — это тот редкий случай, когда синтаксический сахар одновременно делает код **короче**, **безопаснее** и **читабельнее**.
