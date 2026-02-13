#swift #collections #sequence #indexing #search
# firstIndex, firstIndex(of:) и firstIndex(where:) в Swift

Методы семейства `firstIndex` позволяют найти **индекс первого элемента**, который либо равен заданному значению, либо удовлетворяет заданному условию. Это один из самых часто используемых инструментов при работе с упорядоченными коллекциями в [[Swift]].

Все эти методы:

- возвращают **опциональный индекс** → `Index?` (обычно `Int?` для массивов)
- останавливают поиск **при первом совпадении** (early return)
- работают **только на типах**, реализующих протокол [[Collection]] ([[Array]], ArraySlice, ContiguousArray, [[String]], [[Data]], [[Range]] и многие другие)

## Основные варианты (актуально на 2026 год)

```swift
// 1. Поиск по точному равенству (требует Equatable)
func firstIndex(of element: Element) -> Index?
    where Element: Equatable

// 2. Поиск по условию (самый универсальный)
func firstIndex(where predicate: (Element) throws -> Bool) rethrows -> Index?

// 3. Поиск по ключевому пути (KeyPath) — с Swift 5.2+
func firstIndex<T: Equatable>(of element: T, by keyPath: KeyPath<Element, T>) -> Index?
```

## Когда какой метод использовать

| Метод                          | Требование к Element     | Когда использовать                                      | Пример сценария                              |
|--------------------------------|---------------------------|----------------------------------------------------------|----------------------------------------------|
| `firstIndex(of:)`              | `Equatable`               | Ищем точное совпадение с конкретным значением            | Найти позицию конкретного ID, строки, enum   |
| `firstIndex(where:)`           | Нет требований            | Ищем по любому условию, свойству, вычислению             | Первый элемент > 100, первый активный, etc.  |
| `firstIndex(of:by:)` (key path)| `Equatable` на значении   | То же, что `of:`, но через свойство                      | Первый пользователь с name == "Anna"         |
| `firstIndex(of:)` на String    | —                         | Поиск первого вхождения символа / подстроки              | Первая запятая, первый пробел                |

## Примеры использования

### 1. Классический поиск точного значения

```swift
let ids = [1001, 1005, 1012, 1005, 1020]

if let index = ids.firstIndex(of: 1005) {
    print("Первое вхождение 1005 на позиции \(index)")  // → 1
}
```

### 2. Поиск по условию — самый частый сценарий

```swift
struct Task {
    let id: Int
    let title: String
    let priority: Int
    let isCompleted: Bool
    let dueDate: Date?
}

let tasks: [Task] = // ...

// Первый просроченный таск
if let index = tasks.firstIndex(where: { task in
    task.dueDate ?? .distantFuture < Date() && !task.isCompleted
}) {
    // можно пометить как urgent
    tasks[index].priority = 10
}
```

### 3. firstIndex + KeyPath (очень читаемо)

```swift
let users = [
    User(id: 1, name: "Anna", age: 28),
    User(id: 2, name: "Ben",  age: 19),
    User(id: 3, name: "Clara", age: 34)
]

// Первый пользователь старше 30 лет
if let idx = users.firstIndex(where: { $0.age > 30 }) {
    print("Старше 30: \(users[idx].name)")  // Clara
}

// То же самое через key path (Swift 5.2+)
if let idx = users.firstIndex(of: 30, by: \.age, where: >) {
    // или просто
    let adultIndex = users.firstIndex { $0[keyPath: \.age] >= 18 }
}
```

### 4. firstIndex в [[String]]

```swift
let sentence = "Swift is fast, safe, and expressive."

if let commaIndex = sentence.firstIndex(of: ",") {
    let beforeComma = sentence[..<commaIndex]
    print(beforeComma)  // "Swift is fast"
}

if let spaceIndex = sentence.firstIndex(where: \.isWhitespace) {
    // первый пробел
}
```

### 5. Поиск в массиве структур по вложенному свойству

```swift
struct OrderItem {
    let productID: String
    let quantity: Int
}

struct Order {
    let id: String
    let items: [OrderItem]
}

let orders: [Order] = // ...

// Найти первый заказ, содержащий продукт "P-12345"
if let orderIndex = orders.firstIndex(where: { order in
    order.items.contains { $0.productID == "P-12345" }
}) {
    print("Найден в заказе \(orders[orderIndex].id)")
}
```

### 6. Антипаттерн — никогда так не делайте

```swift
// ❌ Очень плохо: полный проход + лишняя аллокация
let wrongIndex = tasks.filter { $0.priority > 5 }.first.map { tasks.firstIndex(of: $0)! }

// ✓ Правильно и эффективно
let rightIndex = tasks.firstIndex { $0.priority > 5 }
```

## Сравнение производительности

| Метод                     | Сложность     | Остановка при первом совпадении | Аллокации | Рекомендация                     |
|---------------------------|---------------|----------------------------------|-----------|-----------------------------------|
| `firstIndex(of:)`         | O(n)          | Да                               | Нет       | Когда есть Equatable              |
| `firstIndex(where:)`      | O(n)          | Да                               | Нет       | Универсальный выбор №1            |
| `filter {}.firstIndex`    | O(n) + O(n)   | Нет                              | Да (массив)| Никогда не использовать           |
| `firstIndex` + `contains` | O(n) × 2      | —                                | Нет       | Лучше один where                  |

## Полезные комбинации (2025–2026)

```swift
// Удаление первого элемента, удовлетворяющего условию
if let idx = items.firstIndex(where: { $0.isInvalid }) {
    items.remove(at: idx)
}

// Замена первого найденного
if let idx = products.firstIndex(of: targetProduct) {
    products[idx] = updatedProduct
}

// Получить индекс и сразу элемент
if let idx = users.firstIndex(where: { $0.id == targetID }),
   let user = users[safe: idx] {   // с помощью safe subscript
    // работаем с user
}
```

## Полезные расширения (очень популярны в проектах)

```swift
extension Collection {
    func firstIndex<T>(matching value: T, by keyPath: KeyPath<Element, T>) -> Index?
    where T: Equatable {
        firstIndex { $0[keyPath: keyPath] == value }
    }
}

// Использование:
let idx = employees.firstIndex(matching: "Engineering", by: \.department)
```

`firstIndex` и `firstIndex(where:)` — это те методы, которые делают код **быстрее**, **безопаснее** и **читабельнее** по сравнению с ручным циклом `for` или `filter(...).first`.

**Главное правило 2026 года**:
- Хотите индекс → `firstIndex(where:)`
- Хотите элемент → `first(where:)`
- Хотите проверить наличие → `contains(where:)`
- Никогда не соединяйте `filter` и `firstIndex` / `first`
