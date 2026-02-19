**`Sequence`** — это фундаментальный протокол в Swift, который определяет типы, элементы которых **можно перебирать по одному за раз** (итерировать).

Все стандартные коллекции Swift соответствуют `Sequence`:

- `Array`
- `Set`
- `Dictionary`
- `String`
- `Range`
- `ClosedRange`
- `StrideThrough`, `StrideTo`, `StrideThrough` и т.д.

> Проще говоря: если тип можно использовать в цикле `for-in`, значит он соответствует `Sequence`.

### 1. Основное требование протокола (2026 актуально)

```swift
public protocol Sequence {
    associatedtype Element
    associatedtype Iterator : IteratorProtocol
    
    func makeIterator() -> Iterator
    
    // ... + множество default-имплементаций
}
```

Главное — метод `makeIterator()`, который возвращает **итератор** (объект, умеющий выдавать элементы по одному через `next()`).

### 2. Почему Sequence важен (ключевые преимущества)

| Возможность                              | Что даёт в 2026 году                                      | Пример |
|------------------------------------------|-----------------------------------------------------------|--------|
| `for-in` цикл                            | Самый читаемый способ перебора                             | `for item in collection { ... }` |
| `map`, `filter`, `reduce`, `compactMap`  | Функциональный стиль обработки                            | `numbers.filter { $0 > 0 }.map { $0 * 2 }` |
| `lazy` обработка                         | Экономия памяти для больших/бесконечных последовательностей | `1...1_000_000.lazy.filter { $0 % 2 == 0 }` |
| `allSatisfy`, `contains`, `min`, `max`   | Удобные агрегатные операции                               | `users.allSatisfy { $0.isActive }` |
| `joined`, `flatMap`, `zip`               | Комбинирование последовательностей                        | `zip(names, ages)` |
| Пользовательские бесконечные последовательности | Генераторы, стримы, Fibonacci, случайные числа и т.д.     | `FibonacciSequence()` |

### 3. Самые важные идиомы Sequence в 2026

#### 3.1 Базовый перебор

```swift
let fruits = ["яблоко", "банан", "апельсин"]

for fruit in fruits {
    print(fruit)
}
```

#### 3.2 Ленивая обработка (самый популярный паттерн для больших данных)

```swift
let hugeRange = 1...1_000_000

let evenDoubled = hugeRange.lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .prefix(10)  // берём только первые 10

for value in evenDoubled {
    print(value) // 4, 8, 12, ..., 40
}
```

→ память не расходуется на весь миллион элементов

#### 3.3 Пользовательский Sequence (бесконечный генератор)

```swift
struct FibonacciSequence: Sequence, IteratorProtocol {
    private var a = 0
    private var b = 1
    
    mutating func next() -> Int? {
        let next = a
        (a, b) = (b, a + b)
        return next
    }
    
    func makeIterator() -> FibonacciSequence {
        self
    }
}

for num in FibonacciSequence() {
    if num > 1000 { break }
    print(num) // 0, 1, 1, 2, 3, 5, 8, ...
}
```

#### 3.4 Sequence + async/await (Swift 5.5+)

```swift
struct AsyncNumbers: AsyncSequence {
    typealias Element = Int
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var current = 0
        
        mutating func next() async -> Int? {
            try? await Task.sleep(nanoseconds: 1_000_000_000) // 1 сек
            current += 1
            return current <= 5 ? current : nil
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        AsyncIterator()
    }
}

Task {
    for await number in AsyncNumbers() {
        print("Получено:", number)
    }
}
```

### 4. Sequence vs Collection — главное отличие

| Характеристика                  | Sequence                                      | Collection (наследует Sequence)               |
|---------------------------------|-----------------------------------------------|-----------------------------------------------|
| Доступ по индексу               | Нет                                           | Да (`collection[index]`)                      |
| `count`                         | Нет (может быть бесконечным)                  | Да                                            |
| Многократный перебор            | Не гарантируется                              | Гарантируется                                 |
| Примеры                         | `AsyncSequence`, генераторы, `AnySequence`    | `Array`, `Set`, `Dictionary`, `String`        |

**Правило 2026**:  
Если нужен только однократный перебор (или ленивость) → достаточно `Sequence`.  
Если нужен индекс, `count`, многократный проход → нужен `Collection`.

### 5. Лучшие практики Sequence в Swift 2026

- **Используй `lazy`** перед `map`/`filter` на больших или бесконечных последовательностях  
- **Создавай свои Sequence** через `AnyIterator` или реализацию `IteratorProtocol`  
- **Для бесконечных последовательностей** — всегда добавляй `prefix(_:)` или `take(while:)`  
- **В async** — используй `AsyncSequence` и `for await`  
- **Не храни** результат `lazy` последовательности в свойстве без необходимости — пусть остаётся ленивым  
- **Swift 6 strict concurrency** — `Sequence` полностью безопасен, но итераторы должны учитывать акторы  
- **Документируйте** — пиши комментарий «Sequence — ленивая генерация чисел Фибоначчи до лимита»

**Короткий девиз 2026**:
> `Sequence` — это всё, что можно **перебрать по одному элементу**.  
> В 2026 году:  
> - `for-in` — базовый способ  
> - `lazy` + `map/filter/reduce` — для экономии памяти  
> - пользовательские Sequence — для генераторов и бесконечных потоков  
> - `AsyncSequence` — для асинхронных потоков  
> Это **основа** функционального и ленивого программирования в Swift.

Удачи с элегантным и эффективным перебором данных в твоём коде! 🔄