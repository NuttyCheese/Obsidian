**`switch`** — это оператор ветвления, который **сравнивает значение с набором паттернов** и выполняет соответствующий блок кода.

- Поддерживает **любые типы, которые соответствуют [[Equatable]]**
    
- Каждое значение должно быть **обработано**, иначе требуется `default`
    
- Поддерживает **pattern matching, where условия, диапазоны и перечисления**
    

> Проще говоря: `switch` = «выбираем ветку кода в зависимости от значения».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Case**|Один вариант в switch с соответствующим блоком кода|
|**Default**|Обязительный кейс, если не все значения обработаны|
|**Pattern Matching**|Сравнение значения с шаблоном, включая диапазоны и enum|
|**Fallthrough**|Позволяет переходить к следующему case (редко используется)|
|**Where Clause**|Дополнительное условие для case|

---

## 3. Основной синтаксис

```swift
let number = 2

switch number {
case 1:
    print("One")
case 2:
    print("Two")
case 3:
    print("Three")
default:
    print("Other")
}
```

- `number` сравнивается с каждым кейсом
    
- `default` обязателен, если не все значения перечислены
    

---

## 4. Примеры от простого к сложному

### Пример 1. [[enum]] + switch

```swift
enum Direction {
    case north, south, east, west
}

let dir = Direction.north

switch dir {
case .north:
    print("Go up")
case .south:
    print("Go down")
case .east:
    print("Go right")
case .west:
    print("Go left")
}
```

- Enum отлично подходит для switch, **default не нужен**, если все кейсы перечислены
    

---

### Пример 2. Диапазоны

```swift
let score = 85

switch score {
case 0..<50:
    print("Fail")
case 50..<70:
    print("Pass")
case 70..<90:
    print("Good")
case 90...100:
    print("Excellent")
default:
    print("Invalid score")
}
```

- Можно использовать **диапазоны** с `<`, `...` и `..<`
    

---

### Пример 3. Where clause

```swift
let number = 8

switch number {
case let x where x % 2 == 0:
    print("Even number")
case let x where x % 2 != 0:
    print("Odd number")
default:
    break
}
```

- `where` добавляет **условие для кейса**
    

---

### Пример 4. [[tuple]] + switch

```swift
let point = (2, 3)

switch point {
case (0, 0):
    print("Origin")
case (let x, 0):
    print("X-axis: \(x)")
case (0, let y):
    print("Y-axis: \(y)")
case (let x, let y):
    print("Point at (\(x), \(y))")
}
```

- Можно использовать **кортежи и деструктуризацию**
    

---

### Пример 5. Pattern Matching с [[Optional]]

```swift
let value: Int? = 5

switch value {
case .some(let x):
    print("Value is \(x)")
case .none:
    print("Value is nil")
}
```

- Используется для **опциональных значений**
    

---

## 5. Особенности switch

1. **Полное покрытие кейсов** — без `default` не компилируется для неполного перечисления
    
2. Поддерживает **pattern matching, диапазоны, кортежи, where**
    
3. Не требует `break` в конце кейса (по умолчанию **не проваливается**)
    
4. `fallthrough` можно использовать для перехода к следующему кейсу
    
5. Может работать с **[[enum]], [[Int]], [[String]], Optional, tuple, кастомными типами**
    

---

## 6. Итог

- **switch** = мощный оператор ветвления с pattern matching
    
- Используется для **enum, чисел, диапазонов, кортежей, опционалов**
    
- Полезен, когда **if/else слишком громоздкий**
    
- Поддерживает **where, fallthrough, деструктуризацию**
    

---
