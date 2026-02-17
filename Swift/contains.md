Метод `contains` — один из самых часто используемых в стандартной библиотеке [[Swift]]. Он позволяет быстро проверить, **присутствует ли** элемент (или элементы, удовлетворяющие условию) в коллекции или последовательности.

Метод существует в **нескольких перегрузках**, и понимание разницы между ними критически важно для правильного и эффективного использования.

## Основные перегрузки метода contains

1. **`contains(_ element: Element) ->` [[Bool]]**
   - Доступен, когда `Element`: [[Equatable]]
   - Проверяет **точное равенство** с переданным элементом
   - Самый быстрый и удобный вариант для примитивов, строк, структур/классов с `Equatable`

2. **`contains(where predicate: (Element) throws -> Bool) rethrows -> Bool`**
   - Доступен всегда (на [[Sequence 1]])
   - Проверяет, **существует ли хотя бы один элемент**, для которого замыкание возвращает `true`
   - Идеален, когда элементы **не Equatable** или нужно сложное условие

3. **`contains(_ other: some Sequence<Element>) -> Bool`** (iOS 16+, macOS 13+)
   - Проверяет, содержит ли коллекция **подпоследовательность** (не обязательно непрерывную)

4. Специфичные для `String`: `contains(_ substring: String)`, `contains<Character>`, `contains(where:)`

5. Для [[Set]] и [[Dictionary]]: `contains(_:)` работает в **O(1)** благодаря хэшированию

## Сигнатуры (основные)

```swift
// Для Array, ContiguousArray, etc. когда Element: Equatable
func contains(_ element: Element) -> Bool

// Универсальная версия (Sequence)
func contains(where predicate: (Element) throws -> Bool) rethrows -> Bool

// Подпоследовательность (с iOS 16 / Swift 5.7+)
func contains<S: Sequence>(_ other: S) -> Bool where S.Element == Element
```

## Примеры использования

### 1. Простая проверка с Equatable

```swift
let numbers = [1, 3, 5, 7, 9, 11]
print(numbers.contains(5))          // true
print(numbers.contains(6))          // false

let names = ["Anna", "Ben", "Clara"]
print(names.contains("Ben"))        // true
```

### 2. contains(where:) — самый мощный и универсальный вариант

```swift
struct User {
    let id: Int
    let age: Int
    let isActive: Bool
}

let users = [
    User(id: 1, age: 23, isActive: true),
    User(id: 2, age: 31, isActive: false),
    User(id: 3, age: 19, isActive: true)
]

// Проверка сложного условия
let hasAdult = users.contains { $0.age >= 18 }              // true
let hasActiveTeen = users.contains { $0.age < 20 && $0.isActive }  // true

// Если тип не Equatable — только contains(where:)
enum Status { case pending, active, failed }
let statuses: [Status] = [.pending, .active, .failed]
print(statuses.contains(.active))                           // Ошибка компиляции!
print(statuses.contains { $0 == .active })                  // true
```

### 3. contains в Set и Dictionary — O(1)

```swift
let ids: Set<Int> = [1001, 1005, 1012, 1020]
print(ids.contains(1005))           // true, очень быстро

let scores: [String: Int] = ["Anna": 95, "Ben": 82, "Clara": 100]
print(scores.keys.contains("Ben"))     // true, O(1)
print(scores.values.contains(100))     // true, но O(n) — линейный поиск
```

### 4. contains подпоследовательности (с [[iOS]] 16 / macOS 13 / Swift 5.7+)

```swift
let sequence = [1, 2, 3, 4, 5, 6]

// Проверяем наличие подпоследовательности (не обязательно подряд)
print(sequence.contains([2, 4, 6]))     // true
print(sequence.contains([6, 5, 4]))     // false
```

### 5. contains в [[String]]

```swift
let text = "The quick brown fox jumps over the lazy dog"

// Простые проверки
print(text.contains("fox"))             // true
print(text.contains("cat"))             // false

// contains с Character
print(text.contains("🐶" as Character)) // false

// contains(where:) для сложных условий
let hasUppercase = text.contains { $0.isUppercase }  // true
let hasDigit = text.contains { $0.isNumber }         // false
```

### 6. Реальные сценарии

```swift
// Фильтрация некорректных ответов от API
let responses = ["200 OK", "404 Not Found", "500 Server Error", "200 OK"]
let hasError = responses.contains { $0.hasPrefix("5") }     // true

// Проверка прав доступа
let userRoles: Set<String> = ["admin", "editor", "viewer"]
let canEdit = userRoles.contains { $0 == "admin" || $0 == "editor" }  // true

// Проверка наличия хотя бы одного валидного email
let emails = ["anna@example.com", "invalid", "ben@company.org", ""]
let hasValidEmail = emails.contains { $0.contains("@") && $0.contains(".") }  // true
```

### 7. Производительность — важные нюансы

| Коллекция             | `contains(_:)` (Equatable) | `contains(where:)` | Примечание                     |
| --------------------- | -------------------------- | ------------------ | ------------------------------ |
| [[Array]]             | O(n)                       | O(n)               | Линейный поиск                 |
| [[Set]]               | O(1)                       | O(n)               | Хэш-поиск для точного элемента |
| Dictionary.keys       | O(1)                       | O(n)               | Хэш-поиск по ключу             |
| [[Dictionary]].values | O(n)                       | O(n)               | Нет хэширования по значению    |
| [[String]]            | O(n)                       | O(n)               | Поиск подстроки или символа    |

**Совет**: если часто проверяете наличие элемента — используйте `Set` вместо `Array`.

## Полезные паттерны и антипаттерны

```swift
// Хорошо: ранний выход при первом совпадении
let hasSpecial = items.contains { $0.isSpecial }

// Плохо: не нужно превращать в массив
let _ = items.filter { $0.isSpecial }.isEmpty    // → лишняя аллокация

// Очень плохо: forced unwrap после contains
if items.contains(where: { $0.id == targetId }) {
    let item = items.first { $0.id == targetId }!   // → дублирование поиска
}

// Лучше:
if let item = items.first(where: { $0.id == targetId }) {
    // используем item
}
```

## Итог — когда какой вариант использовать

- Знаете точное значение и `Element: Equatable` → `contains(_:)`
- Нужно условие / сравнение / свойства → `contains(where:)`
- Хотите проверить наличие подпоследовательности → `contains<S: Sequence>(_:)`
- Часто ищете по значению → переложите данные в `Set`
- Работаете с [[API]] / [[JSON]] → `contains(where:)` почти всегда ваш лучший друг

Метод `contains` — простой, но невероятно мощный инструмент, который делает код Swift выразительным и читаемым. Используйте правильную перегрузку — и ваш код станет быстрее и чище.
