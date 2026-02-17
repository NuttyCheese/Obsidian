**`Float`** — это числовой тип с плавающей точкой, использующий **32 бита** для хранения значения.

- Представляет **десятичные дроби и целые числа**
    
- Менее точный, чем [[Double]]
    
- Используется для **экономии памяти** при больших объёмах чисел или когда высокая точность не нужна
    

> Проще говоря: Float = «число с плавающей точкой, но с меньшей точностью, чем Double».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Floating Point**|Число с десятичной точкой, может быть дробным|
|**Precision**|Точность хранения (Float = ~6–7 значимых цифр)|
|**Range**|Максимальное и минимальное представимое значение|
|**Scientific Notation**|Можно записывать числа в экспоненциальной форме (1.0e3 = 1000)|

---

## 3. Основной синтаксис

```swift
var pi: Float = 3.14159
var radius = 10.0 as Float // Явное преобразование
```

- Можно **явно указать тип** (`: Float`)
    
- Можно преобразовать `Double` или [[Int]] в `Float`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простые операции

```swift
var a: Float = 5.5
var b: Float = 2.0

let sum = a + b       // 7.5
let difference = a - b // 3.5
let product = a * b    // 11.0
let quotient = a / b   // 2.75

print(sum, difference, product, quotient)
```

- Поддерживаются стандартные арифметические операции
    

---

### Пример 2. Округление

```swift
var number: Float = 3.14159
let rounded = round(number)   // 3.0
let ceilValue = ceil(number)  // 4.0
let floorValue = floor(number) // 3.0

print(rounded, ceilValue, floorValue)
```

- `round()`, `ceil()`, `floor()` работают с Float
    

---

### Пример 3. Scientific Notation

```swift
let large: Float = 1.2e3   // 1.2 * 10^3 = 1200.0
let small: Float = 3.4e-3  // 0.0034

print(large, small)
```

- Удобно для больших и маленьких чисел
    

---

### Пример 4. Math functions

```swift
import Foundation

let value: Float = 16.0
let squareRoot = sqrt(value)      // 4.0
let power = pow(value, 2)         // 256.0
let sine = sin(Float.pi / 2)      // 1.0

print(squareRoot, power, sine)
```

- Используем математические функции через `Foundation`
    
- Обратите внимание на **приведение констант типа Double → Float**
    

---

### Пример 5. Conversion & Optional Float

```swift
let intValue = 42
let floatValue = Float(intValue) // 42.0

let stringValue = "3.14"
if let parsed = Float(stringValue) {
    print(parsed * 2) // 6.28
}
```

- Преобразование **Int → Float**
    
- Парсинг из строки через `Float(string)` → Optional Float
    

---

## 5. Особенности Float

1. **32-битная точность** (~6–7 значимых цифр)
    
2. Поддерживает арифметику и математические функции
    
3. Можно использовать с **[[Optional]]**
    
4. Позволяет экономить память по сравнению с Double
    
5. Поддерживает **scientific notation** и стандартные функции
    

---

## 6. Итог

- **Float** = число с плавающей точкой одинарной точности
    
- Используется, когда **не требуется высокая точность**
    
- Поддерживает:
    
    - Арифметику (+, -, *, /)
        
    - Округление (round, ceil, floor)
        
    - Scientific notation (1.0e3)
        
    - Math functions (sqrt, pow, sin, cos)
        
    - Конвертацию с Int и [[String]]
        

---
