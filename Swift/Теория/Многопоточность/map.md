`map` — это, пожалуй, **самая фундаментальная** и самая часто используемая функция высшего порядка в [[Swift]]. Она лежит в основе почти любого функционального преобразования коллекций.

Метод `map` применяет заданное замыкание (или функцию) **к каждому элементу** коллекции и возвращает **новую коллекцию** той же длины, но с элементами нового типа (или того же).

## Основные сигнатуры (Swift 6+)

```swift
// Самая распространённая
func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]

// Для коллекций, сохраняющих тип (String, ArraySlice и др.)
func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]
    // но иногда сохраняется исходный тип, если это RangeReplaceableCollection

// С KeyPath (Swift 5.2+)
func map<T>(_ keyPath: KeyPath<Element, T>) -> [T]
```

## Что важно понимать про map

- **Всегда создаёт новую коллекцию** — исходная не меняется
- **Сохраняет порядок** элементов
- **Длина результата = длине исходной коллекции**
- **Может бросать ошибки** (throwing-замыкание)
- **Ленив** при использовании `.lazy.map { … }`
- **Тип результата** обычно `[T]`, но для некоторых типов ([[String]], [[Set]]) может быть тот же тип

## Примеры использования

### 1. Простое преобразование

```swift
let numbers = [1, 2, 3, 4, 5]
let squared   = numbers.map { $0 * $0 }           // [1, 4, 9, 16, 25]
let doubled   = numbers.map { $0 * 2 }            // [2, 4, 6, 8, 10]
let strings   = numbers.map(String.init)          // ["1", "2", "3", "4", "5"]
```

### 2. Работа со структурами / классами

```swift
struct User {
    let id: Int
    let name: String
    let age: Int
    let isActive: Bool
}

let users: [User] = // ...

let names       = users.map { $0.name }
let capitalized = users.map { $0.name.uppercased() }
let greetings   = users.map { "Hello, \($0.name)!" }
```

### 3. map через KeyPath (самый популярный современный стиль)

```swift
let userNames    = users.map(\.name)                // [String]
let userAges     = users.map(\.age)                 // [Int]
let activeFlags  = users.map(\.isActive)            // [Bool]

// Комбинированный пример
let descriptions = users.map { "\($0.name), \($0.age) лет" }
```

### 4. map на [[String]]

```swift
let greeting = "Swift is awesome!"

let uppercased = greeting.map { $0.uppercased() }   // ["S", "W", "I", "F", "T", " ", …]
let asciiCodes = greeting.map(\.asciiValue)         // [83, 119, 105, 102, 116, …] типа [UInt8?]
let onlyLetters = greeting.map { $0.isLetter ? $0 : "_" }.joined()
```

### 5. map + опционалы (часто комбинируют с [[compactMap]])

```swift
let optionalNumbers: [Int?] = [1, nil, 3, nil, 5]
let strings = optionalNumbers.map { $0.map(String.init) ?? "—" }
// → ["1", "—", "3", "—", "5"]
```

### 6. map в цепочках (очень частый паттерн)

```swift
let result = users
    .filter { $0.isActive }
    .sorted { $0.age < $1.age }
    .map { user in
        "\(user.name) (\(user.age))"
    }
    .joined(separator: ", ")
```

### 7. Ленивый map (когда важен порядок вычислений)

```swift
let hugeData = (1...1_000_000).lazy
    .map { expensiveTransformation($0) }    // вычисление откладывается
    .filter { $0 > threshold }
    .prefix(100)

for value in hugeData {
    // здесь происходит реальное вычисление только для нужных элементов
}
```

## Когда использовать map, а когда другие методы

| Задача                                    | Метод(ы)                                                  | Почему                               |
| ----------------------------------------- | --------------------------------------------------------- | ------------------------------------ |
| Преобразовать каждый элемент              | `map`                                                     | Основной инструмент                  |
| Преобразовать и убрать `nil`              | `compactMap`                                              | `map` оставит опционалы              |
| Преобразовать и расплющить                | [[flatMap]]                                               | `map` не расплющивает                |
| Преобразовать и отфильтровать             | `map` + [[filter]] или наоборот                           | порядок влияет на производительность |
| Выполнить сайд-эффекты                    | [[forEach]]                                               | `map` создаёт ненужный массив        |
| Найти первый элемент после преобразования | `first(where:)` + transform или `compactMap {}.`[[first]] | —                                    |

## Антипаттерны (не делайте так)

```swift
// ❌ Плохо: map вместо forEach для сайд-эффектов
_ = users.map { user in
    print(user.name)            // создаётся ненужный массив строк
}

// ✓ Лучше
users.forEach { print($0.name) }

// ❌ Плохо: map + force unwrap
let ids = optionalUsers.map { $0!.id }  // опасно

// ✓ Безопасно
let ids = optionalUsers.compactMap { $0?.id }
```

## Полезные комбинации (2025–2026)

```swift
// Топ-5 самых активных пользователей с форматированным выводом
let topUsers = users
    .filter(\.isActive)
    .sorted { $0.lastActive > $1.lastActive }
    .prefix(5)
    .map { "\($0.name) — last seen \($0.lastActive.formatted())" }
    .joined(separator: "\n")

// Преобразование enum в строку
enum Status { case pending, active, failed }
let statusStrings = statuses.map(\.rawValue)     // если есть rawValue
```

`map` — это сердце функционального программирования в Swift.  
Он делает код **короче**, **безопаснее** (иммутабельность), **выразительнее** и позволяет легко комбинировать операции в цепочки.

**Главное правило**:  
Если вы хотите **изменить каждый элемент** и получить **новую коллекцию** той же длины — **это почти всегда `map`**.
