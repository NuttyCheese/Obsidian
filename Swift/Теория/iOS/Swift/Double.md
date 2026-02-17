**`Double`** — это **числовой тип с плавающей точкой**, который использует **64 бита** для хранения значения.

- Представляет **десятичные дроби и целые числа**
    
- Более точный, чем [[Float]] (который использует 32 бита)
    
- Используется для **математических операций, вычислений с высокой точностью, финансовых и научных задач**
    

> Проще говоря: Double = «число с плавающей точкой, точнее, чем Float».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Floating Point**|Число с десятичной точкой, может быть дробным|
|**Precision**|Точность хранения (Double = 15–16 значимых цифр)|
|**Range**|Максимальное и минимальное представимое значение|
|**Scientific Notation**|Можно записывать числа в экспоненциальной форме (1.0e3 = 1000)|

---

## 3. Основной синтаксис

```swift
var pi: Double = 3.14159
var radius = 10.0 // Swift выводит Double автоматически
```

- Можно **явно указать тип** (`: Double`) или использовать type inference
    
- Можно присваивать целые числа (`10`) — они автоматически преобразуются
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простые операции

```swift
var a: Double = 5.5
var b: Double = 2.0

let sum = a + b       // 7.5
let difference = a - b // 3.5
let product = a * b    // 11.0
let quotient = a / b   // 2.75

print(sum, difference, product, quotient)
```

- Поддерживаются стандартные арифметические операции
    

---

### Пример 2. Округление и форматирование

```swift
var number: Double = 3.14159
let rounded = round(number)   // 3.0
let ceilValue = ceil(number)  // 4.0
let floorValue = floor(number) // 3.0

print(rounded, ceilValue, floorValue)
```

- `round()`, `ceil()`, `floor()` для округления
    

---

### Пример 3. Scientific Notation

```swift
let large: Double = 1.2e6   // 1.2 * 10^6 = 1200000.0
let small: Double = 3.4e-3  // 0.0034

print(large, small)
```

- Удобно для **очень больших и очень маленьких чисел**
    

---

### Пример 4. Math functions

```swift
import Foundation

let value: Double = 16.0
let squareRoot = sqrt(value)      // 4.0
let power = pow(value, 2)         // 256.0
let sine = sin(Double.pi / 2)     // 1.0

print(squareRoot, power, sine)
```

- Поддерживаются **математические функции** через `Foundation`
    

---

### Пример 5. Conversion & Optional Double

```swift
let intValue = 42
let doubleValue = Double(intValue) // 42.0

let stringValue = "3.14"
if let parsed = Double(stringValue) {
    print(parsed * 2) // 6.28
}
```

- Преобразование **Int → Double**
    
- Парсинг из строки через `Double(string)` → Optional Double
    

---

## 5. Особенности Double

1. **64-битная точность** — лучше, чем Float
    
2. Поддерживает **арифметику, округление, функции**
    
3. Может быть **[[Optional]]** для парсинга или неинициализированных значений
    
4. Используется для **финансовых, научных и инженерных вычислений**
    
5. Поддерживает **scientific notation** и математические операции
    

---

## 6. Итог

- **Double** = число с плавающей точкой двойной точности
    
- Используется для **дробных и больших чисел, научных вычислений**
    
- Поддерживает:
    
    - Арифметику (+, -, *, /)
        
    - Округление (round, ceil, floor)
        
    - Scientific notation (1.0e3)
        
    - Math functions (sqrt, pow, sin, cos)
        
    - Конвертацию с [[Int]] и [[String]]
        

---
