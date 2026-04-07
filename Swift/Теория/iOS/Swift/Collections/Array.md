#swift #array #collection #data-structures #performance #value-type

---

## Array в Swift

### Определение
**Array** — это упорядоченная коллекция элементов одного типа, доступ к которым осуществляется по целочисленному индексу. Swift-массивы хранят элементы в строгом порядке, позволяют дубликаты и обеспечивают быстрый доступ по индексу.

[[Swift]] Array является **[[Value Type]]** (копируется при присваивании), но использует **[[Copy-on-Write]] (COW)** для оптимизации производительности.

### Зачем это знать iOS-разработчику?
1.  **Самая часто используемая коллекция** — практически в любом приложении есть массивы.
2.  **Производительность:** Важно понимать стоимость операций (доступ, вставка, удаление).
3.  **Copy-on-Write:** Понимание COW помогает избежать неожиданного копирования больших массивов.
4.  **Совместимость с UI:** [[UITableView]], [[UICollectionView]], `SwiftUI List` работают с массивами.
5.  **Фундамент:** Многие алгоритмы и структуры данных строятся на массивах.

---

### Основные характеристики

| Характеристика        | Array     | [[Set Collection\|Set]] | [[Dictionary]]  |
| --------------------- | --------- | ----------------------- | --------------- |
| **Упорядоченность**   | Да        | Нет                     | Нет             |
| **Дубликаты**         | Да        | Нет                     | Ключи уникальны |
| **Доступ по индексу** | Да (O(1)) | Нет                     | Нет             |
| **Поиск элемента**    | O(n)      | O(1)                    | O(1) (по ключу) |
| **Требования к типу** | Нет       | [[Hashable]]            | Key: `Hashable` |

---

### Создание Array

#### 1. **Пустой массив**

```swift
var emptyArray = [Int]()
var anotherEmpty: [String] = []
```

#### 2. **С литералом**

```swift
let fruits = ["Apple", "Banana", "Orange"]
let numbers = [1, 2, 3, 4, 5]
```

#### 3. **С повторяющимися значениями**

```swift
let zeros = Array(repeating: 0, count: 5)   // [0, 0, 0, 0, 0]
let array = [Int](repeating: 42, count: 3)  // [42, 42, 42]
```

#### 4. **Из диапазона**

```swift
let range = Array(1...5)      // [1, 2, 3, 4, 5]
let rangeStep = Array(stride(from: 0, to: 10, by: 2))  // [0, 2, 4, 6, 8]
```

#### 5. **Из другого массива**

```swift
let original = [1, 2, 3]
let copy = original  // копирование при изменении (COW)
```

---

### Основные операции

#### Доступ к элементам

```swift
let fruits = ["Apple", "Banana", "Orange"]

// По индексу
let first = fruits[0]        // "Apple"
let last = fruits[fruits.count - 1]  // "Orange"

// first, last (безопасные)
let safeFirst = fruits.first  // Optional("Apple")
let safeLast = fruits.last    // Optional("Orange")

// withUnsafeBufferPointer (для низкоуровневой работы)
fruits.withUnsafeBufferPointer { buffer in
    print(buffer.baseAddress!)  // указатель на начало
}
```

#### Добавление элементов

```swift
var numbers = [1, 2, 3]

// В конец
numbers.append(4)              // [1, 2, 3, 4]
numbers.append(contentsOf: [5, 6])  // [1, 2, 3, 4, 5, 6]

// В начало
numbers.insert(0, at: 0)       // [0, 1, 2, 3, 4, 5, 6]

// В середину
numbers.insert(99, at: 3)      // [0, 1, 2, 99, 3, 4, 5, 6]

// + оператор
let newArray = numbers + [7, 8]  // создаёт новый массив
```

#### Удаление элементов

```swift
var numbers = [1, 2, 3, 4, 5]

// По индексу
let removed = numbers.remove(at: 2)  // removed = 3, numbers = [1, 2, 4, 5]

// Последний
let last = numbers.removeLast()      // last = 5, numbers = [1, 2, 4]

// Первый
let first = numbers.removeFirst()    // first = 1, numbers = [2, 4]

// Все
numbers.removeAll()                  // []
numbers.removeAll(keepingCapacity: true)  // очистка, но память не освобождается

// Удаление по диапазону
var items = [0, 1, 2, 3, 4, 5]
items.removeSubrange(1...3)          // [0, 4, 5]
```

#### Изменение элементов

```swift
var numbers = [1, 2, 3, 4, 5]

// По индексу
numbers[0] = 10          // [10, 2, 3, 4, 5]

// По диапазону
numbers[1...3] = [20, 30, 40]  // [10, 20, 30, 40, 5]
numbers[1...2] = [100]         // [10, 100, 40, 5] (замена 2 элементов на 1)

// swapAt
numbers.swapAt(0, 1)     // [100, 10, 40, 5]
```

---

### Доступ к подмассивам

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8]

// Subarray (не копирует данные!)
let slice = numbers[2...5]     // [3, 4, 5, 6] (ArraySlice)
let prefix = numbers.prefix(3) // [1, 2, 3]
let suffix = numbers.suffix(2) // [7, 8]

// Преобразование slice в Array
let newArray = Array(slice)    // [3, 4, 5, 6] (копирование)
```

**Важно:** `ArraySlice` — это представление части массива без копирования данных. Он делит буфер с оригинальным массивом. При длительном хранении лучше преобразовать в `Array`.

---

### Итерация

```swift
let fruits = ["Apple", "Banana", "Orange", "Mango"]

// for-in
for fruit in fruits {
    print(fruit)
}

// forEach
fruits.forEach { fruit in
    print(fruit)
}

// С индексом
for (index, fruit) in fruits.enumerated() {
    print("\(index): \(fruit)")
}

// reversed
for fruit in fruits.reversed() {
    print(fruit)  // Mango, Orange, Banana, Apple
}

// stride
for i in stride(from: 0, to: fruits.count, by: 2) {
    print(fruits[i])  // Apple, Orange
}
```

---

### Поиск и проверка

```swift
let numbers = [10, 20, 30, 40, 50]

// contains
let hasTwenty = numbers.contains(20)     // true
let hasEven = numbers.contains { $0 % 2 == 0 }  // true

// firstIndex
let indexOfThirty = numbers.firstIndex(of: 30)  // Optional(2)
let indexOfEven = numbers.firstIndex { $0 % 2 == 0 }  // Optional(0)

// lastIndex
let lastIndexOfEven = numbers.lastIndex { $0 % 2 == 0 }  // Optional(4)

// allSatisfy
let allEven = numbers.allSatisfy { $0 % 2 == 0 }  // false
```

---

### Сортировка

```swift
var numbers = [3, 1, 4, 1, 5, 9, 2]

// sorted (возвращает новый массив)
let ascending = numbers.sorted()                // [1, 1, 2, 3, 4, 5, 9]
let descending = numbers.sorted(by: >)          // [9, 5, 4, 3, 2, 1, 1]

// sort (на месте)
numbers.sort()                                   // numbers = [1, 1, 2, 3, 4, 5, 9]

// reverse
numbers.reverse()                                // numbers = [9, 5, 4, 3, 2, 1, 1]

// shuffle
numbers.shuffle()                                // случайный порядок
```

---

### Трансформация

```swift
let numbers = [1, 2, 3, 4, 5]

// map
let squares = numbers.map { $0 * $0 }           // [1, 4, 9, 16, 25]

// compactMap (удаляет nil)
let strings = ["1", "a", "2", "b", "3"]
let ints = strings.compactMap { Int($0) }       // [1, 2, 3]

// flatMap (разворачивает вложенные массивы)
let nested = [[1, 2], [3, 4], [5]]
let flat = nested.flatMap { $0 }                // [1, 2, 3, 4, 5]

// filter
let evens = numbers.filter { $0 % 2 == 0 }      // [2, 4]

// reduce
let sum = numbers.reduce(0, +)                  // 15
let product = numbers.reduce(1, *)              // 120
```

---

### Производительность

| Операция | Сложность | Примечание |
|----------|-----------|------------|
| `append` | O(1) амортизированная | Может быть O(n) при расширении буфера |
| `insert(at:)` | O(n) | Сдвиг элементов |
| `remove(at:)` | O(n) | Сдвиг элементов |
| `removeLast()` | O(1) | |
| `removeFirst()` | O(n) | Сдвиг элементов |
| `access via index` | O(1) | Прямой доступ |
| `first` / `last` | O(1) | |
| `contains` | O(n) | |
| `sort` | O(n log n) | |
| `reversed()` | O(1) | Создаёт ленивую коллекцию |

---

### Под капотом: устройство Array

Swift Array — это **структура, управляющая буфером в куче через Copy-on-Write (COW)**.

#### 1. **Структура памяти**

```text
Array (value type)
┌─────────────────────────────┐
│ storage: BufferPointer      │ →  Heap buffer
│ count: Int                  │     ┌─────┬─────┬─────┬─────┐
│ capacity: Int               │     │  1  │  2  │  3  │  4  │
└─────────────────────────────┘     └─────┴─────┴─────┴─────┘
```

- `Array` сам по себе — маленькая структура на стеке (три слова: storage, count, capacity)
- Реальные элементы хранятся в **непрерывном буфере в куче** (как C-массив)
- Несколько массивов могут ссылаться на **один и тот же буфер** (COW)

#### 2. **Copy-on-Write (COW)**

```swift
var a = [1, 2, 3]   // создаётся буфер
var b = a           // b разделяет буфер с a (без копирования!)
b.append(4)         // здесь происходит копирование буфера
```

**Алгоритм COW:**
1.  При изменении массива проверяется уникальность буфера (`isKnownUniquelyReferenced`)
2.  Если буфер разделяется → создаётся копия → изменения применяются к копии
3.  Если буфер уникален → изменения прямо в нём

#### 3. **Рост массива (capacity)**

```swift
var numbers = [Int]()
print(numbers.capacity)  // 0 (зависит от версии Swift)

numbers.append(1)
print(numbers.capacity)  // 2 (начальная ёмкость)

numbers.append(2)
print(numbers.capacity)  // 2 (ёмкость не меняется)

numbers.append(3)
print(numbers.capacity)  // 4 (удваивается)
```

**Стратегия роста:** ёмкость увеличивается в геометрической прогрессии (обычно в 2 раза), чтобы амортизировать стоимость вставки O(1).

#### 4. **`reserveCapacity`**

```swift
var numbers = [Int]()
numbers.reserveCapacity(1000)  // выделяет буфер на 1000 элементов заранее

for i in 0..<1000 {
    numbers.append(i)  // без лишних реаллокаций
}
```

#### 5. **ArraySlice и COW**

```swift
var a = [1, 2, 3, 4, 5]
let slice = a[1...3]   // slice разделяет буфер с a

a[0] = 100              // копирования нет — slice всё ещё ссылается на старый буфер
print(slice)            // [2, 3, 4] (старые данные)
```

**Важно:** При изменении оригинала slice продолжает ссылаться на старый буфер (COW).

#### 6. **Маленькие массивы**

Для очень маленьких массивов Swift может использовать **оптимизацию inline storage** — хранить элементы прямо в структуре `Array`, не выделяя heap.

---

### Сравнение с Objective-C

| Характеристика      | Swift Array          | NSArray / NSMutableArray |
| ------------------- | -------------------- | ------------------------ |
| **Тип**             | [[Value type]]       | [[Reference type]]       |
| **Copy-on-Write**   | Да                   | Нет                      |
| **Типизация**       | [[Generic]], строгая | Id (любой объект)        |
| **Может содержать** | Любые типы           | Только объекты           |
| **Модификация**     | Через методы         | addObject, removeObject  |
| **Совместимость**   | bridge к [[NSArray]] | bridge к Array           |

---

### Bridging между Array и NSArray

```swift
import Foundation

// Swift Array → NSArray
let swiftArray = [1, 2, 3]
let nsArray = swiftArray as NSArray

// NSArray → Swift Array
let nsArray: NSArray = [1, 2, 3]
let swiftArray = nsArray as? [Int] ?? []
```

**Примечание:** Swift Array автоматически мостится к NSArray, когда это необходимо (например, при вызове [[UIKit]] [[API]]). Это может вызвать неожиданное копирование.

---

### Лучшие практики

#### 1. **Резервируйте ёмкость для больших массивов**

```swift
// ❌ Плохо — много реаллокаций
var numbers = [Int]()
for i in 0..<10000 {
    numbers.append(i)
}

// ✅ Хорошо
var numbers = [Int]()
numbers.reserveCapacity(10000)
for i in 0..<10000 {
    numbers.append(i)
}
```

#### 2. **Используйте `isEmpty` вместо проверки `count`**

```swift
// ❌ Плохо
if array.count > 0 { }

// ✅ Хорошо
if array.isEmpty { }
```

#### 3. **Избегайте `removeFirst()` для больших массивов**

```swift
// ❌ Плохо — O(n) на каждое удаление
var queue = [1, 2, 3, 4, 5]
let first = queue.removeFirst()

// ✅ Хорошо — используйте другой тип (например, IndexSet)
// или обрабатывайте массив с индексами
```

#### 4. **Проверяйте границы при доступе по индексу**

```swift
// ❌ Плохо — crash при выходе за границы
let value = array[index]

// ✅ Хорошо — безопасный доступ
if array.indices.contains(index) {
    let value = array[index]
}

// ✅ Или через optional
let value = array[safe: index]  // кастомный сабскрипт
```

#### 5. **Используйте `lazy` для цепочек операций**

```swift
// ❌ Плохо — создаёт промежуточные массивы
let result = array.map { $0 * 2 }.filter { $0 > 10 }.prefix(5)

// ✅ Хорошо — ленивые вычисления
let result = array.lazy.map { $0 * 2 }.filter { $0 > 10 }.prefix(5)
```

#### 6. **Безопасный сабскрипт**

```swift
extension Array {
    subscript(safe index: Int) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
}

let array = [1, 2, 3]
let value = array[safe: 5]  // nil, без crash
```

---

### Короткое правило

> **Array** — упорядоченная коллекция с доступом по индексу за O(1).  
> Используйте для последовательных данных, где важен порядок и быстрый доступ.  
> COW делает копирование дешёвым, но вставка/удаление в середине — O(n).

---

### Итог

**Array** в Swift:

1.  **Value type с COW** — копируется при изменении
2.  **Элементы упорядочены** и могут повторяться
3.  **Доступ по индексу O(1)**
4.  **Вставка/удаление в начале/середине O(n)** — сдвиг элементов
5.  **append в конец O(1) амортизированная**
6.  **Хранит элементы в непрерывном буфере** (как C-массив)
7.  **Поддерживает COW** для экономии памяти
8.  **Умеет резервировать ёмкость** (`reserveCapacity`)

Понимание устройства и производительности Array помогает писать эффективный код и избегать неожиданных задержек.