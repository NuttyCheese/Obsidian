#swift #protocol #strideable #stride #range #sequence #numeric

---
### Определение
**`Strideable`** — это протокол в [[Swift]], который определяет тип, значения которого могут быть перемещены на определенную величину (шаг) с использованием операции сложения и разности . Типы, соответствующие `Strideable`, поддерживают арифметику с дискретными шагами, что позволяет эффективно генерировать последовательности значений через `stride(from:to:by:)` и `stride(from:through:by:)`.

Протокол наследует от `Comparable`, поэтому все `Strideable` типы также являются сравниваемыми. Основные встроенные типы, соответствующие `Strideable`: все целочисленные ([[Int]], `UInt`, `Int8` и т.д.) и типы с плавающей точкой ([[Float]], [[Double]], [[CGFloat]]) .

### Зачем это знать iOS-разработчику?
1.  **Генерация последовательностей:** Создание диапазонов значений с определенным шагом.
2.  **Арифметика с дискретными шагами:** Удобная работа с числовыми последовательностями.
3.  **Кастомные типы:** Возможность реализовать `Strideable` для своих типов (например, для работы с датами, индексами, градусами).
4.  **Понимание Swift Standard Library:** Знание протоколов помогает лучше понимать стандартную библиотеку.
5.  **Алгоритмы:** Многие алгоритмы используют `Strideable` для эффективной работы с диапазонами.

---

### Протокол Strideable

```swift
protocol Strideable<Stride> : Comparable {
    associatedtype Stride : SignedNumeric
    
    func distance(to other: Self) -> Stride
    func advanced(by n: Stride) -> Self
}
```

#### Требования протокола:

| Требование | Описание |
|------------|----------|
| **`Stride`** | Ассоциированный тип, представляющий величину шага. Должен соответствовать `SignedNumeric` (поддерживать отрицательные значения). |
| **`distance(to:)`** | Возвращает расстояние от текущего значения до указанного. Результат может быть отрицательным. |
| **`advanced(by:)`** | Возвращает новое значение, смещенное на указанное количество шагов. |

#### Наследование:
- **`Comparable`:** Все `Strideable` типы могут сравниваться (`<`, `>`, `==`).

---

### Встроенные Strideable типы

| Тип                       | Stride           | Пример                            |
| ------------------------- | ---------------- | --------------------------------- |
| `Int`                     | `Int`            | `1.advanced(by: 2)` → `3`         |
| `Double`                  | `Double`         | `1.5.advanced(by: 2.5)` → `4.0`   |
| `CGFloat`                 | `CGFloat`        | `10.0.advanced(by: -5.0)` → `5.0` |
| [[Date]] ([[Foundation]]) | [[TimeInterval]] | `date.addingTimeInterval(60)`     |

---

### Примеры использования

#### 1. **Базовые операции**

```swift
// Целые числа
let start = 10
let end = start.advanced(by: 5)
print(end)  // 15

let distance = start.distance(to: 20)
print(distance)  // 10

// С плавающей точкой
let pi: Double = 3.14159
let halfPi = pi.advanced(by: -1.5708)
print(halfPi)  // ≈ 1.57079
```

#### 2. **Функции stride**

```swift
// stride(from:to:by:) — не включает конечное значение
for value in stride(from: 0, to: 10, by: 2) {
    print(value)  // 0, 2, 4, 6, 8
}

// stride(from:through:by:) — включает конечное значение
for value in stride(from: 0, through: 10, by: 2) {
    print(value)  // 0, 2, 4, 6, 8, 10
}

// Обратный шаг
for value in stride(from: 10, to: 0, by: -2) {
    print(value)  // 10, 8, 6, 4, 2
}
```

#### 3. **Работа с Double**

```swift
let values = Array(stride(from: 0.0, to: 1.0, by: 0.1))
print(values)  // [0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9]

// Внимание: из-за неточности Double, лучше использовать целые числа
let preciseValues = (0..<10).map { Double($0) / 10 }
```

#### 4. **Создание последовательностей**

```swift
// Генерация четных чисел
let evenNumbers = stride(from: 0, to: 20, by: 2)
for number in evenNumbers {
    print(number)  // 0, 2, 4, ..., 18
}

// Создание массива
let squares = stride(from: 0, through: 100, by: 10).map { $0 * $0 }
print(squares)  // [0, 100, 400, 900, 1600, 2500, 3600, 4900, 6400, 8100, 10000]
```

---

### Реализация Strideable для пользовательских типов

#### 1. **Тип для градусов**

```swift
struct Degrees: Strideable {
    var value: Double
    
    // Strideable requirements
    typealias Stride = Double
    
    func distance(to other: Degrees) -> Double {
        return other.value - self.value
    }
    
    func advanced(by n: Double) -> Degrees {
        return Degrees(value: self.value + n)
    }
}

// Использование
let start = Degrees(value: 0)
let end = Degrees(value: 180)

let step = start.distance(to: end)  // 180
let half = start.advanced(by: 90)   // 90°

for angle in stride(from: start, through: end, by: 45) {
    print(angle.value)  // 0, 45, 90, 135, 180
}
```

#### 2. **Тип для дат (нативный подход)**

```swift
import Foundation

struct CustomDate: Strideable, Comparable {
    let date: Date
    
    typealias Stride = TimeInterval
    
    func distance(to other: CustomDate) -> TimeInterval {
        return other.date.timeIntervalSince(self.date)
    }
    
    func advanced(by n: TimeInterval) -> CustomDate {
        return CustomDate(date: self.date.addingTimeInterval(n))
    }
    
    static func < (lhs: CustomDate, rhs: CustomDate) -> Bool {
        return lhs.date < rhs.date
    }
    
    static func == (lhs: CustomDate, rhs: CustomDate) -> Bool {
        return lhs.date == rhs.date
    }
}

// Использование
let now = CustomDate(date: Date())
let tomorrow = now.advanced(by: 86400)  // +1 день
let distance = now.distance(to: tomorrow)  // 86400.0
```

#### 3. **Тип для индексов**

```swift
struct StepIndex: Strideable {
    var step: Int
    
    typealias Stride = Int
    
    func distance(to other: StepIndex) -> Int {
        return other.step - self.step
    }
    
    func advanced(by n: Int) -> StepIndex {
        return StepIndex(step: self.step + n)
    }
}

extension StepIndex: Comparable {
    static func < (lhs: StepIndex, rhs: StepIndex) -> Bool {
        return lhs.step < rhs.step
    }
}

// Использование
let startIndex = StepIndex(step: 0)
let endIndex = StepIndex(step: 10)

for index in stride(from: startIndex, to: endIndex, by: 2) {
    print(index.step)  // 0, 2, 4, 6, 8
}
```

---

### Strideable и [[Range]]

```swift
// Range<Int> работает только с целыми числами
let intRange = 0..<10

// Для Strideable типов можно использовать stride
let doubleRange = stride(from: 0.0, to: 10.0, by: 0.5)

// Преобразование в массив
let doubleArray = Array(doubleRange)
```

---

### Связь со Stride

Swift также предоставляет **`Stride`** и **`StrideThrough`** структуры для представления последовательностей:

```swift
// StrideTo — to (не включая конечное значение)
let strideTo = stride(from: 0, to: 10, by: 2)
print(type(of: strideTo))  // StrideTo<Int>

// StrideThrough — through (включая конечное значение)
let strideThrough = stride(from: 0, through: 10, by: 2)
print(type(of: strideThrough))  // StrideThrough<Int>
```

---

### Производительность

```swift
import Darwin

func measure(_ name: String, _ block: () -> Void) {
    let start = mach_absolute_time()
    block()
    let end = mach_absolute_time()
    var info = mach_timebase_info()
    mach_timebase_info(&info)
    let elapsed = (end - start) * UInt64(info.numer) / UInt64(info.denom)
    print("\(name): \(elapsed / 1_000_000) ms")
}

measure("Stride for loop") {
    var sum = 0
    for i in stride(from: 0, to: 10_000_000, by: 1) {
        sum += i
    }
}

measure("Range for loop") {
    var sum = 0
    for i in 0..<10_000_000 {
        sum += i
    }
}

// Примерный результат: оба цикла имеют схожую производительность
```

---

### Лучшие практики

#### 1. **Используйте stride для дискретных последовательностей**

```swift
// ✅ Хорошо
for angle in stride(from: 0, to: 360, by: 45) {
    // обработка углов
}

// ❌ Плохо для больших диапазонов с малым шагом
for angle in stride(from: 0, to: 360, by: 0.001) {
    // может создать миллионы элементов
}
```

#### 2. **Для Double используйте целочисленный подход при возможности**

```swift
// ❌ Может дать неточности
let values = stride(from: 0.0, to: 1.0, by: 0.1)

// ✅ Более точно
let values = (0..<10).map { Double($0) / 10 }
```

#### 3. **Реализуйте Strideable для кастомных числовых типов**

```swift
struct Celsius: Strideable {
    var value: Double
    typealias Stride = Double
    
    func distance(to other: Celsius) -> Double {
        return other.value - self.value
    }
    
    func advanced(by n: Double) -> Celsius {
        return Celsius(value: self.value + n)
    }
}
```

---

### Короткое правило

> **`Strideable`** — протокол для типов, поддерживающих арифметику с дискретными шагами.  
> Используйте `stride(from:to:by:)` и `stride(from:through:by:)` для генерации последовательностей.  
> Реализуйте для кастомных числовых типов (углы, индексы, координаты).

### Итог

**`Strideable`** — важный протокол Swift Standard Library:

1.  **Позволяет** генерировать последовательности с заданным шагом.
2.  **Требует** реализации `distance(to:)` и `advanced(by:)`.
3.  **Наследует** от [[Comparable]].
4.  **Встроенные типы:** `Int`, `Double`, `Float`, `CGFloat`.
5.  **Применение:** циклы с шагом, создание массивов, работа с дискретными величинами.
6.  **Кастомные типы:** полезно для градусов, дат, индексов, координат.

Понимание `Strideable` помогает эффективно работать с последовательностями чисел и создавать удобные [[API]] для кастомных типов .