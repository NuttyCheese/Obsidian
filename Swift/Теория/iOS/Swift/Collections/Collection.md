**Collection** — это фундаментальный протокол в [[Swift]], который лежит в основе всех упорядоченных коллекций языка: массивов, строк, словарей, множеств, диапазонов и пользовательских типов.

Он расширяет протокол **[[protocol Sequence|Sequence]]** и добавляет возможность **доступа по индексу**, **подсчёта элементов** и **безопасной итерации**.

В 2026 году `Collection` — это один из самых важных протоколов стандартной библиотеки, и почти весь код, работающий с последовательностями, опирается именно на него.

### 1. Что делает тип Collection-ом (минимальные требования)

Чтобы тип соответствовал протоколу `Collection`, он должен реализовать:

```swift
protocol Collection: Sequence {
    associatedtype Element
    associatedtype Index: Comparable
    
    var startIndex: Index { get }
    var endIndex: Index { get }
    
    func index(after i: Index) -> Index
    
    subscript(position: Index) -> Element { get }
}
```

После этого автоматически становятся доступны:

- `count`, `isEmpty`
- `first`, `last`
- `indices` (диапазон всех индексов)
- for-in итерация
- [[map]], [[filter]], [[reduce]], [[sort|sorted]], [[compactMap]] и многие другие методы

### 2. Сравнение Collection с Sequence (самое частое заблуждение)

| Характеристика        | Sequence                                    | Collection                                                | Когда использовать в 2026              |
| --------------------- | ------------------------------------------- | --------------------------------------------------------- | -------------------------------------- |
| Доступ по индексу     | Нет (только последовательный проход)        | Да (startIndex, subscript)                                | Collection — если нужен индекс         |
| count / isEmpty       | Нет (нужно пройтись)                        | Да (O(1) в большинстве случаев)                           | Collection — для быстрого подсчёта     |
| [[first]] / last      | Да (O(1) в массивах, O(n) в других)         | Да (O(1) гарантировано)                                   | Collection — если важен O(1) доступ    |
| Многократная итерация | Да (но может быть дорогой)                  | Да (и эффективнее)                                        | Collection — для многократного прохода |
| Примеры типов         | `AnySequence`, `StrideSequence`, генераторы | [[Array]], [[String]], [[Set Collection]], [[Dictionary]], [[Range]] | —                                      |

**Правило 2026**:  
Если тебе нужен **доступ по индексу** или **быстрый count** — требуй `Collection`.  
Если достаточно **однократного прохода** (map, reduce, for-in) — достаточно `Sequence`.

### 3. Самые популярные типы, conforming к Collection (2026)

| Тип                      | Index тип      | Элемент тип                | Особенности 2026                     | Самый частый сценарий |
| ------------------------ | -------------- | -------------------------- | ------------------------------------ | --------------------- |
| `Array<Element>`         | `Int`          | `Element`                  | [[Copy-on-Write]], contiguous memory | Всё подряд            |
| `String`                 | `String.Index` | [[Character]]              | Unicode-safe, grapheme clusters      | Текст, эмодзи         |
| `Set<Element>`           | Специальный    | `Element`                  | Неупорядоченный, хэш-таблица         | Уникальные элементы   |
| `Dictionary<Key, Value>` | Специальный    | `(key: Key, value: Value)` | Неупорядоченный, хэш-таблица         | Ключ-значение         |
| `Range<Bound>`           | `Bound`        | `Bound`                    | Ленивый диапазон                     | Индексы, итерации     |
| `ClosedRange<Bound>`     | `Bound`        | `Bound`                    | Закрытый диапазон                    | Валидация границ      |
| `ClosedRange<Int>`       | `Int`          | [[Int]]                    | Самый частый диапазон                | Циклы, слайсы         |

### 4. Полный пример кастомной коллекции (2026 стиль)

```swift
struct SimpleLinkedList<Element> {
    private class Node {
        var value: Element
        var next: Node?
        init(value: Element) { self.value = value }
    }
    
    private var head: Node?
    private var tail: Node?
    private(set) var count = 0
}

// MARK: - Collection conformance
extension SimpleLinkedList: Collection {
    var startIndex: Int { 0 }
    var endIndex: Int { count }
    
    func index(after i: Int) -> Int {
        i + 1
    }
    
    subscript(position: Int) -> Element {
        var current = head
        for _ in 0..<position {
            current = current?.next
        }
        return current!.value
    }
}

// MARK: - Mutable operations
extension SimpleLinkedList {
    mutating func append(_ element: Element) {
        let node = Node(value: element)
        if let tail {
            tail.next = node
        } else {
            head = node
        }
        tail = node
        count += 1
    }
}

// Использование
var list = SimpleLinkedList<Int>()
list.append(10)
list.append(20)
list.append(30)

for value in list {
    print(value) // 10, 20, 30
}

print(list[1]) // 20
print(list.count) // 3
print(list.isEmpty) // false
```

### 5. Лучшие практики Collection в Swift 2026

- **Требуй Collection**, если нужен доступ по индексу или count — это даёт больше возможностей  
- **Используй Sequence**, если достаточно однократного прохода (map, reduce, for-in)  
- **Не полагайся на Int как индекс** — у `String` индекс — `String.Index`, у `Dictionary` — специальный тип  
- **Для кастомных коллекций** — реализуй `Collection` минимально (startIndex, endIndex, index(after:), subscript) — остальное дастся бесплатно  
- **Copy-on-Write** — в `Array`, `String`, `Set`, `Dictionary` — копирование происходит только при мутации  
- **Swift 6 strict concurrency** — коллекции полностью `Sendable`, если их элементы `Sendable`  
- **Документируйте** — пиши комментарий «[User] — коллекция пользователей, conforming к Collection»

**Короткий девиз 2026**:
> `Collection` — это когда тебе нужна **упорядоченная последовательность с доступом по индексу**.  
> В 2026 году это **основа** всех массивов, строк, диапазонов, множеств и словарей.  
> Требуй `Collection`, если нужен `count`, индексация или многократный проход — это даёт компилятору и тебе больше безопасности и оптимизаций.
