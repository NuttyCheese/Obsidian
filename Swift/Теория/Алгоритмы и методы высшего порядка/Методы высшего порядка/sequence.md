#swift #collections #sequence #iteration #enumeration
# sequence(…) — ленивые последовательности без написания итератора

Функции `sequence` — это **фабричные методы** из стандартной библиотеки [[Swift]], которые позволяют создать **ленивую последовательность** ([[lazy]] sequence) всего за одно замыкание, без необходимости писать собственные типы `Sequence` и `IteratorProtocol`.

Они возвращают тип `UnfoldSequence<Element, State>` — ленивую, однократную последовательность, элементы которой вычисляются **только по мере обращения** к ним.

## Две основные формы

```swift
// Вариант 1 — самый популярный
func sequence<A>(first: A, next: @escaping (A) → A?) → UnfoldSequence<A, A>

// Вариант 2 — более мощный, с произвольным состоянием
func sequence<State, Element>(
    state: State,
    next: @escaping (inout State) → Element?
) → UnfoldSequence<Element, State>
```

## Ключевые характеристики

- **Ленивость** — элементы не генерируются заранее
- **Однократный проход** — последовательность можно пройти только один раз
- **Завершение** — когда замыкание `next` возвращает [[nil]]
- **Тип возвращаемого значения** — `UnfoldSequence<Element, State>`
- **Нет хранения** — внутри нет массива, только текущее состояние и замыкание

## Часть 1. sequence(first:next:)

Простейшая и самая часто используемая форма.

### Как работает внутренне

```swift
var current = first
while let nextValue = next(current) {
    yield current          // ← текущий элемент отдаётся потребителю
    current = nextValue
}
// если next вернул nil → конец
```

### Типичные сценарии использования

```swift
// 1. Бесконечная арифметическая последовательность
let naturals     = sequence(first: 1)  { $0 + 1 }
let evenNumbers  = sequence(first: 2)  { $0 + 2 }
let countdown    = sequence(first: 10) { $0 > 0 ? $0 - 1 : nil }

// 2. Геометрическая прогрессия
let powersOf2    = sequence(first: 1) { $0 &* 2 }   // с защитой от переполнения можно &*

// 3. Фибоначчи (простой, но не самый эффективный вариант)
let fibSimple = sequence(first: 0) { prev in
    // здесь нужно помнить два предыдущих → неудобно
    // лучше использовать sequence(state:)
}

// 4. Итерация по датам
import Foundation

let tomorrow = sequence(first: Date()) { date in
    Calendar.current.date(byAdding: .day, value: 1, to: date)
}

// 5. Обход связного списка (очень частый кейс)
class Node<T> {
    var value: T
    var next: Node<T>?
}

let head: Node<Int>? = …
let values = sequence(first: head) { $0?.next }
    .compactMap { $0?.value }
```

## Часть 2. sequence(state:next:) — настоящий король

Здесь состояние может быть **произвольным** (кортеж, структура, класс, массив, словарь…).

Замыкание получает **inout**-доступ к состоянию → можно менять всё, что угодно.

### Классические примеры

```swift
// 1. Настоящий Фибоначчи (самый чистый вариант)
let fibonacci = sequence(state: (0, 1)) { state -> Int? in
    let (a, b) = state
    state = (b, a &+ b)           // &+ — защита от переполнения
    return a < 1_000_000 ? a : nil
}

Array(fibonacci.prefix(15))
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
```

```swift
// 2. Разбиение на чанки фиксированного размера
func chunks<T>(of size: Int, in array: [T]) -> some Sequence<[T]> {
    sequence(state: 0) { index -> [T]? in
        guard index < array.count else { return nil }
        let end = min(index + size, array.count)
        let chunk = Array(array[index..<end])
        index = end
        return chunk
    }
}

let data = Array(1...17)
for chunk in chunks(of: 5, in: data) {
    print(chunk)
}
// [1,2,3,4,5], [6,7,8,9,10], [11,12,13,14,15], [16,17]
```

```swift
// 3. Генератор псевдослучайных чисел (линейный конгруэнтный)
struct LCG {
    var state: UInt64 = 1
    mutating func next() -> UInt64 {
        state = state &* 6364136223846793005 &+ 1
        return state
    }
}

let randoms = sequence(state: LCG()) { rng -> UInt64? in
    rng.next()
}
```

```swift
// 4. Парсинг строки посимвольно с состоянием
let text = "Hello, 世界! Swift 6.0"

let charactersWithInfo = sequence(state: text.startIndex) { index -> (Character, String.Index)? in
    guard index < text.endIndex else { return nil }
    let char = text[index]
    let nextIndex = text.index(after: index)
    index = nextIndex
    return (char, index)
}
```

## Когда sequence(…) — лучший выбор

| Сценарий                                  | Почему sequence лучше альтернативы                     |
|-------------------------------------------|----------------------------------------------------------|
| Бесконечная / очень длинная последовательность | Не создаёт массив в памяти заранее                       |
| Генерация по правилу «следующий от предыдущего» | Код в 3–5 строк вместо отдельного итератора             |
| Ленивые чанки, батчи, пагинация           | Эффективно по памяти                                     |
| Итерация по связным структурам            | Node → next → sequence(first: head) { $0?.next }         |
| Симуляция потоков, генераторов             | Очень близко к генераторам в других языках               |
| Фибоначчи, геометрические прогрессии      | Особенно с состоянием-кортежем                           |

## Антипаттерны и ловушки

```swift
// ❌ Плохо — sequence не предназначен для многократного прохода
let seq = sequence(first: 1) { $0 + 1 }
print(Array(seq.prefix(5)))     // [1,2,3,4,5]
print(Array(seq.prefix(5)))     // [] ← уже закончился!

// ✓ Правильно — если нужен многократный доступ → материализуйте
let numbers = Array(sequence(first: 1) { $0 + 1 }.prefix(100))
```

```swift
// ❌ Неэффективно — лучше flatMap или joined
let nested = [[1,2], [3], [4,5,6]]
let flatWrong = sequence(state: nested) { arrs -> Int? in … } // сложно и ненужно
```

## Итоговая шпаргалка 2026 года

```swift
// Простая цепочка значений
sequence(first: начальное) { предыдущее → следующее? }

// Всё, что требует памяти состояния
sequence(state: (…, …)) { (inout состояние) → Элемент? }

// Самые частые паттерны
sequence(first: 0)  { $0 + 1 }                    // счётчик
sequence(first: 1)  { $0 &* 2 }                   // степени двойки
sequence(state: (0,1)) { … }                      // Фибоначчи
sequence(state: 0) { idx in …; idx += шаг }       // чанки, слайсы
sequence(first: head) { $0?.next }                // связный список
```

`sequence(…)` — это один из самых элегантных инструментов в Swift для создания **ленивых, декларативных, экономных по памяти** потоков данных.

Когда в следующий раз захочешь написать цикл с изменяемыми переменными — спроси себя:  
«А нельзя ли это выразить через `sequence(state: …)`?»  
Очень часто — **можно**, и код станет чище и красивее.
