#swift #standard_library #collection #array #sequence #mutation #element_swap #inplace
# swapAt(_:_:) в Swift

Метод `swapAt(_:_:)` — это один из самых полезных и часто недооцениваемых методов массива в стандартной библиотеке [[Swift]]. Он позволяет **эффективно** поменять местами два элемента массива по их индексам **без создания временных переменных** и без лишних аллокаций.

## Синтаксис

```swift
mutating func swapAt(_ i: Int, _ j: Int)
```

- Требует, чтобы массив был объявлен как [[var]] (изменяемый)
- Работает только на типах, реализующих `MutableCollection` (в первую очередь [[Array]], `ContiguousArray`)
- **Не возвращает** ничего (`Void`)
- **Бросает** `Index out of bounds`, если хотя бы один индекс недопустим

## Почему swapAt лучше, чем ручной обмен

```swift
// Старый, неэффективный и громоздкий способ
var temp = array[i]
array[i] = array[j]
array[j] = temp

// Современный и рекомендуемый способ (с Swift 3+)
array.swapAt(i, j)
```

Преимущества `swapAt`:

- **Безопасность** — проверяет границы автоматически
- **Эффективность** — использует низкоуровневые оптимизации (особенно для ContiguousArray)
- **Читаемость** — намерение обмена видно сразу
- **Нет лишних копий** — особенно важно для больших структур и классов

## Примеры использования

### 1. Простой обмен

```swift
var numbers = [10, 20, 30, 40, 50]
numbers.swapAt(1, 3)
print(numbers)  // [10, 40, 30, 20, 50]
```

### 2. В алгоритмах сортировки (очень частый сценарий)

```swift
func bubbleSort<T: Comparable>(_ array: inout [T]) {
    for i in 0..<array.count {
        for j in 0..<(array.count - 1 - i) {
            if array[j] > array[j + 1] {
                array.swapAt(j, j + 1)
            }
        }
    }
}

var values = [64, 34, 25, 12, 22, 11, 90]
bubbleSort(&values)
print(values)  // [11, 12, 22, 25, 34, 64, 90]
```

### 3. Быстрая сортировка (QuickSort) — классический пример

```swift
func partition<T: Comparable>(_ array: inout [T], low: Int, high: Int) -> Int {
    let pivot = array[high]
    var i = low - 1
    
    for j in low..<high {
        if array[j] <= pivot {
            i += 1
            array.swapAt(i, j)
        }
    }
    array.swapAt(i + 1, high)
    return i + 1
}
```

### 4. Перемешивание массива (Fisher–Yates / современный вариант)

```swift
extension Array {
    mutating func shuffle() {
        guard count > 1 else { return }
        
        for i in stride(from: count - 1, through: 1, by: -1) {
            let j = Int.random(in: 0...i)
            swapAt(i, j)
        }
    }
}

var cards = Array(1...52)
cards.shuffle()
print(cards.prefix(5))  // пример: [17, 42, 8, 31, 50]
```

### 5. Обмен элементов в многомерных структурах (через индексы)

```swift
var matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

// Меняем местами диагональные элементы
matrix[0].swapAt(0, 2)  // первая строка: 3, 2, 1
matrix[2].swapAt(0, 2)  // третья строка: 9, 8, 7
```

### 6. Работа с индексами в цикле (часто встречается)

```swift
var tasks = ["low", "high", "medium", "critical", "low"]

// Переместить все "critical" в начало
var criticalIndex = 0
for i in 0..<tasks.count {
    if tasks[i] == "critical" {
        tasks.swapAt(i, criticalIndex)
        criticalIndex += 1
    }
}
print(tasks)  // ["critical", "low", "medium", "high", "low"] (пример)
```

### 7. Безопасная работа с опциональными индексами

```swift
var items = ["A", "B", "C", "D"]

if let i = items.firstIndex(of: "B"),
   let j = items.firstIndex(of: "D") {
    items.swapAt(i, j)
    print(items)  // ["A", "D", "C", "B"]
}
```

## Важные нюансы и ловушки (2026)

- `swapAt` **не работает** на неизменяемых коллекциях ([[let]])

```swift
let fixed = [1,2,3]
fixed.swapAt(0, 1)  // Ошибка компиляции: Cannot use mutating member on immutable value
```

- Если индексы одинаковые — **ничего не происходит** (безопасно)

```swift
array.swapAt(2, 2)  // ничего не меняется
```

- Работает с **любыми элементами** (не требует [[Equatable]] или [[Comparable]])

- Очень эффективен для **ContiguousArray** (меньше накладных расходов)

- При работе с большими массивами классов/структур **не копирует** объекты (только меняет указатели)

## Когда использовать swapAt

| Сценарий                              | Рекомендация использовать swapAt |
|---------------------------------------|-----------------------------------|
| Реализация сортировок (bubble, quick, insertion) | Да, обязательно |
| Алгоритм Fisher–Yates (shuffle)       | Да, стандартный способ |
| Перестановка элементов по условию     | Да, очень удобно |
| Работа с индексами в циклах           | Да, чище и безопаснее |
| Просто поменять два элемента          | Да, вместо тройного присваивания |
| Нужно сохранить исходный массив       | Нет → используйте `sorted()` или копию |

## Итог — современные рекомендации

- Хотите **поменять два элемента в массиве** → **всегда** `swapAt`
- Пишете **собственную сортировку** или **перемешивание** → `swapAt` — ваш лучший друг
- Работаете с **неизменяемым массивом** → сначала `.sorted()`, либо создайте `var`-копию
- Код должен быть **читаемым** → `swapAt(i, j)` намного понятнее, чем три строки с временной переменной

`swapAt` — маленький метод, но он значительно улучшает читаемость и производительность кода, связанного с перестановками элементов.
