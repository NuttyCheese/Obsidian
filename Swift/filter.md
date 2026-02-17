`filter` — один из самых фундаментальных и часто используемых методов в стандартной библиотеке Swift. Он позволяет **отобрать** из коллекции или последовательности только те элементы, которые удовлетворяют заданному условию, создавая при этом **новую коллекцию** того же типа (или близкого).

Метод существует практически на всех типах, реализующих протокол [[Sequence 1]] (а значит — на [[Collection]], [[Array]], [[Set]], [[Dictionary]], [[String]], [[Range]], пользовательских последовательностях и т.д.).

## Основные сигнатуры (2025–2026)

```swift
// Самая распространённая
func filter(_ isIncluded: (Element) throws -> Bool) rethrows -> [Element]

// Для коллекций с сохранением индексов (если нужно)
func filter(_ isIncluded: (Element) throws -> Bool) rethrows -> Self
    where Self: RangeReplaceableCollection  // Array, ContiguousArray, etc.

// Специфично для String (сохраняет тип String)
extension StringProtocol {
    func filter(_ isIncluded: (Character) throws -> Bool) rethrows -> Self
}
```

## Что важно понимать про filter

- **Не мутирует** исходную коллекцию (иммутабельный стиль)
- **Всегда создаёт новую коллекцию** (даже если результат совпадает с исходной)
- **Ленивость** — в цепочках с [[lazy]] фильтрация откладывается до первого обращения к элементам
- **Ранний выход не происходит** — `filter` всегда проходит **всю** последовательность
- **Порядок сохраняется** (для упорядоченных коллекций: Array, String и т.д.)
- **Тип результата** обычно `[Element]`, но для некоторых типов (String, Set) — тот же тип

## Примеры использования

### 1. Классические примеры

```swift
let numbers = Array(1...20)

let evens          = numbers.filter { $0.isMultiple(of: 2) }           // [2,4,6,...,20]
let aboveTen       = numbers.filter { $0 > 10 }                        // [11,12,...,20]
let multiplesOf3or5 = numbers.filter { $0.isMultiple(of: 3) || $0.isMultiple(of: 5) }
```

### 2. Работа со структурами / классами

```swift
struct Product {
    let name: String
    let price: Double
    let isAvailable: Bool
    let category: Category
}

enum Category: String { case electronics, clothing, books, food }

let products: [Product] = // ...

// Только доступные товары дороже 1000
let expensiveAvailable = products.filter {
    $0.isAvailable && $0.price > 1000
}

// Только электроника
let electronics = products.filter { $0.category == .electronics }

// Без определённой категории (nil-safe)
let uncategorized = products.filter { $0.category == nil }  // если category опционально
```

### 3. filter на String

```swift
let text = "Swift 6.0 released in 2024 – performance is amazing!"

let lettersOnly = text.filter { $0.isLetter }           // "Swiftreleasedinperformanceisamazing"
let digitsOnly  = text.filter { $0.isNumber }           // "602024"
let asciiOnly   = text.filter { $0.isASCII }            // без эмодзи и не-ASCII
let noSpaces    = text.filter { !$0.isWhitespace }     // всё без пробелов
```

### 4. filter на Set и Dictionary

```swift
let activeUserIDs: Set<Int> = [1001, 1005, 1012, 1020, 1050]
let premiumIDs = activeUserIDs.filter { id in
    premiumUsers.contains(id)   // O(1) если premiumUsers — тоже Set
}

// Фильтрация ключей словаря
let scores: [String: Int] = ["Anna": 95, "Ben": 82, "Clara": 100, "David": 67]
let highScores = scores.filter { $0.value >= 90 }
// → [("Anna", 95), ("Clara", 100)]  типа [(key: String, value: Int)]

// Только ключи
let highScorers = scores.filter { $0.value >= 90 }.map(\.key)
// → ["Anna", "Clara"]
```

### 5. filter + key path (очень популярный современный стиль)

```swift
let adults = users.filter(\.age >= 18)
let active  = users.filter(\.isActive)
let recent  = transactions.filter(\.date > oneMonthAgo)
```

### 6. filter в цепочках (очень частый паттерн)

```swift
let result = users
    .filter { $0.isActive }
    .filter { $0.age >= 18 }
    .sorted { $0.name < $1.name }
    .map { "\($0.name) (\($0.age))" }
```

Или ещё лучше — один `filter` с составным условием:

```swift
let activeAdults = users.filter { $0.isActive && $0.age >= 18 }
```

### 7. Ленивый filter (когда важен порядок выполнения)

```swift
let largeImages = imageFiles
    .lazy
    .filter { $0.fileSize > 5_000_000 }     // фильтруем сначала
    .filter { $0.hasAlphaChannel }          // потом это
    .map { $0.thumbnail() }                 // и только потом дорогое преобразование

// Элементы обрабатываются только при итерации — экономим ресурсы
```

## Производительность и важные нюансы (2026)

| Коллекция    | Сложность filter | Примечание                                   |
| ------------ | ---------------- | -------------------------------------------- |
| Array        | O(n)             | Линейный проход                              |
| Set          | O(n)             | Нет хэширования при фильтрации               |
| Dictionary   | O(n)             | Проход по всем парам (key, value)            |
| String       | O(n)             | Работает с [[Character]] / Unicode.Scalar    |
| LazySequence | O(1) (отложено)  | Вычисления происходят только при потреблении |

**Частые ошибки и как их избежать**

```swift
// ❌ Плохо: несколько filter подряд с взаимоисключающими условиями
let a = items.filter { $0.price > 100 }
let b = a.filter { $0.price < 50 }     // всегда пусто!

// ✓ Лучше: одно условие
let wrongPrice = items.filter { $0.price > 100 && $0.price < 50 } // пусто, но честно

// ❌ Дорого: filter + first где можно first(where:)
if let firstMatch = items.filter({ $0.id == target }).first { ... }
// лучше:
if let firstMatch = items.first(where: { $0.id == target }) { ... }
```

## Полезные комбинации и паттерны 2025–2026

```swift
// Топ-5 самых дорогих доступных товаров
let top5 = products
    .filter(\.isAvailable)
    .sorted { $0.price > $1.price }
    .prefix(5)

// Удаление дубликатов по ключевому полю (через Set + filter)
let uniqueByEmail = allUsers
    .reduce(into: Set<String>()) { seen, user in
        if seen.insert(user.email).inserted {
            // первый раз встретили
        }
    }
    // или короче:
    .filter { Set($0.map(\.email)).contains($1.email) } // не оптимально

// Более эффективно:
let uniqueUsers = Array(Dictionary(grouping: allUsers, by: \.email).values.map { $0[0] })
```

`filter` — это сердце функционального стиля в Swift. Он прост, предсказуем и чрезвычайно универсален. Главное — помнить:

- Используй **один** `filter` вместо нескольких, если возможно
- Для поиска первого элемента лучше `first(where:)`
- Для ленивых вычислений добавляй `.lazy`
- При работе с большими данными думай о `lazy` + `filter` + `map` / `compactMap`
