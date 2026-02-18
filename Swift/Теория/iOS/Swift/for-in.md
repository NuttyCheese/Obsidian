**`for-in`** — это основной цикл в Swift для итерации по элементам коллекции или последовательности.  
Он работает с любым типом, который соответствует протоколу `Sequence` (а чаще всего — `Collection`).

> Проще говоря: `for-in` = «возьми каждый элемент по очереди и сделай с ним что-то».

### 1. Основной синтаксис и варианты (2026 актуально)

```swift
// Классический вариант
for item in collection {
    // item — текущее значение
}

// С индексом (самый частый в реальных проектах)
for (index, item) in collection.enumerated() {
    // index — Int, начиная с 0
}

// Только индексы
for index in collection.indices {
    let item = collection[index]
}

// С where-фильтром (очень читаемо и эффективно)
for item in collection where item > 0 {
    // только положительные элементы
}

// С pattern matching (особенно мощно с enum и Optional)
for case let .success(value) in results {
    // только успешные результаты
}
```

### 2. Самые популярные коллекции, с которыми работает for-in

| Тип коллекции          | Тип элемента         | Индекс тип     | Особенности 2026                             |
| ---------------------- | -------------------- | -------------- | -------------------------------------------- |
| [[Array]]`<T>`         | `T`                  | [[Int]]        | Самый частый, [[COW]], contiguous memory     |
| [[String]]             | [[Character]]        | `String.Index` | Unicode-safe, итерация по графемам           |
| [[Set]]`<T>`           | `T`                  | (внутренний)   | Неупорядоченный, быстрый contains            |
| [[Dictionary]]`<K, V>` | `(key: K, value: V)` | (внутренний)   | Неупорядоченный, доступ по ключу             |
| `ClosedRange<Bound>`   | `Bound`              | `Bound`        | Ленивый диапазон (1...10)                    |
| `StrideThrough<T>`     | `T`                  | `T.Stride`     | Шаг с плавающей точкой (stride(from:to:by:)) |
| Кастомные типы         | `Element`            | `Index`        | Любой тип, реализующий `Collection`          |

### 3. Под капотом: как работает for-in (упрощённо)

```swift
// Компилятор превращает примерно так:
var iterator = collection.makeIterator()
while let item = iterator.next() {
    // тело цикла
}
```

- `makeIterator()` — создаёт итератор один раз перед циклом  
- `next()` — возвращает следующий элемент или `nil` при завершении  
- Итератор **держит ссылку** на коллекцию и текущую позицию  
- Благодаря **[[Copy-On-Write]]** (COW) в `Array`, `String`, `Set`, `Dictionary` — итерация не копирует данные, пока коллекция не изменяется внутри цикла

### 4. Самые полезные идиомы [[for-in]] в 2026 году

#### 4.1. Безопасная итерация с индексом

```swift
for (index, item) in items.enumerated() {
    print("Элемент \(index): \(item)")
}
```

#### 4.2. Фильтрация прямо в цикле (очень читаемо)

```swift
for user in users where user.isActive && user.age >= 18 {
    sendWelcomeEmail(to: user)
}
```

#### 4.3. Работа с Optional элементами

```swift
let optionalNumbers: [Int?] = [1, nil, 3, nil, 5]

for case let number? in optionalNumbers {
    print("Нашли число: \(number)")
}
// Вывод: 1, 3, 5
```

#### 4.4. Рекурсивный обход дерева (с indirect [[enum]])

```swift
indirect enum Tree {
    case empty
    case node(value: Int, children: [Tree])
}

func printTree(_ tree: Tree, indent: String = "") {
    switch tree {
    case .empty:
        break
    case .node(let value, let children):
        print("\(indent)- \(value)")
        for child in children {
            printTree(child, indent: indent + "  ")
        }
    }
}
```

#### 4.5. Итерация по словарю с сортировкой ключей

```swift
let scores = ["Alice": 95, "Bob": 82, "Charlie": 91]

for (name, score) in scores.sorted(by: { $0.value > $1.value }) {
    print("\(name): \(score)")
}
// Charlie: 91
// Alice: 95
// Bob: 82
```

### 5. Лучшие практики for-in в Swift 2026

- **Предпочитай** `where` вместо `if ... continue` — читаемость выше  
- **Используй** `enumerated()` когда нужен индекс  
- **Для Optional** — `for case let ...?` — самый безопасный и красивый способ  
- **Не изменяй коллекцию** внутри цикла по ней — это может привести к краху (особенно Array)  
- **Для больших коллекций** — избегай копирования: не делай `for item in array.map { ... }` — лучше `for item in array { ... }`  
- **Swift 6 strict concurrency** — итерация по `Sendable` коллекциям безопасна  
- **Документируйте** — пиши комментарий «for-in — обработка активных пользователей»

**Короткий девиз 2026**:
> `for-in` — это когда ты говоришь: «возьми каждый элемент и сделай с ним вот это».  
> В 2026 году:  
> - `where` вместо `if continue`  
> - `enumerated()` для индекса  
> - `case let ...?` для Optional  
> - не мутируй коллекцию внутри цикла  
> Это **самый читаемый** и **самый безопасный** способ итерации в Swift.
