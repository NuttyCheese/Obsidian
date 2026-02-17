**`Array`** — это **коллекция элементов одного типа**, которая:

- Хранит **значения в упорядоченном виде**
    
- Поддерживает **индексацию и итерацию**
    
- Может быть **mutable** ([[var]]) или **[[immutable]]** ([[let]])
    
- Является **[[Value Type]]**, то есть копируется при присваивании
    

> Проще говоря: `Array` = «упорядоченный список значений одного типа».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Element**|Тип элементов массива (`Array<Element>`)|
|**Mutable / Immutable**|Можно изменять (`var`) или нет (`let`)|
|**Index**|Позиция элемента в массиве (начинается с 0)|
|**Count**|Количество элементов|
|**Appending / Removing**|Добавление и удаление элементов|
|**Iteration**|Проход по массиву через `for-in`, `forEach` или методы higher-order функций|
|**Higher-order functions**|map, filter, reduce, sorted, compactMap и др.|

---

## 3. Основной синтаксис

```swift
var numbers: [Int] = [1, 2, 3, 4, 5]
let names: [String] = ["Alice", "Bob", "Eve"]
```

- Тип можно указать явно [[Int]], [[String]]
    
- Swift может вывести тип автоматически `var arr = [1, 2, 3]`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Создание и доступ к элементам

```swift
var numbers = [10, 20, 30, 40]
print(numbers[0]) // 10
numbers[1] = 25
print(numbers) // [10, 25, 30, 40]
```

- Доступ по индексу начинается с 0
    
- Изменять элементы можно только если массив mutable (`var`)
    

---

### Пример 2. Добавление и удаление элементов

```swift
var fruits = ["Apple", "Banana"]
fruits.append("Orange")
fruits += ["Mango", "Pineapple"]
fruits.remove(at: 1)
print(fruits) // ["Apple", "Orange", "Mango", "Pineapple"]
```

- Методы: `append`, `+=`, `remove`, `insert`
    

---

### Пример 3. Итерация и [[forEach]]

```swift
let colors = ["Red", "Green", "Blue"]
for color in colors {
    print(color)
}

colors.forEach { print($0) }
```

- Можно использовать **[[for-in]]** или **`forEach`**
    

---

### Пример 4. Higher-order functions

```swift
let numbers = [1, 2, 3, 4, 5]
let squares = numbers.map { $0 * $0 }
let even = numbers.filter { $0 % 2 == 0 }
let sum = numbers.reduce(0, +)

print(squares) // [1, 4, 9, 16, 25]
print(even)    // [2, 4]
print(sum)     // 15
```

- `map` → трансформация
    
- `filter` → фильтрация
    
- `reduce` → свёртка/суммирование
    

---

### Пример 5. Массивы [[any]] и [[AnyObject]]

```swift
var mixed: [Any] = [1, "Hello", 3.14]
mixed.append(Person(name: "Alice")) // можно добавлять любые типы

var objects: [AnyObject] = [Person(name: "Bob"), Person(name: "Eve")]
```

- `Array<Any>` → любой тип
    
- `Array<AnyObject>` → только объекты классов
    

---

### Пример 6. Multidimensional Array

```swift
let matrix: [[Int]] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[1][2]) // 6
```

- Массив массивов → удобен для матриц, таблиц
    

---

### Пример 7. [[Optional]] и [[compactMap]]

```swift
let strings: [String?] = ["1", nil, "3"]
let numbers = strings.compactMap { $0.flatMap(Int.init) }
print(numbers) // [1, 3]
```

- `compactMap` → превращает и фильтрует [[nil]]
    

---

## 5. Особенности Array

1. **Value type** → копируется при присваивании
    
2. **Mutable/Immutable**: `var` можно менять, `let` — нет
    
3. Поддерживает **индексацию, итерацию, сортировку, фильтрацию, [[map]]/[[reduce]]/[[filter]]**
    
4. Может хранить **любой тип**, включая `Any` и `AnyObject`
    
5. Можно делать **многомерные массивы**
    

---

## 6. Итог

- **Array** = упорядоченный список элементов одного типа
    
- Поддерживает **доступ по индексу, добавление/удаление, итерацию, higher-order функции**
    
- Используется для хранения данных, работы с коллекциями, матриц, моделей и др.
    

---
