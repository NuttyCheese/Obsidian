[[Swift]] сообщает, что **вы попытались обратиться к элементу коллекции ([[Array]], [[String]], etc.) по индексу, который находится за пределами допустимого диапазона**.

- Для массивов диапазон индексов — `0..<array.count`.
    
- Ошибка возникает **только во время выполнения**, компилятор не может предсказать её на этапе сборки.
    
- Часто встречается при:
    
    1. Циклах с неправильной границей.
        
    2. Пустых массивах.
        
    3. Доступе к String через индекс без проверки границ.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Доступ к элементу массива**

```swift
let numbers = [1, 2, 3]
print(numbers[3]) // ❌ Index out of range
```

- Допустимые индексы: 0, 1, 2.
    

---

**Пример 2: Пустой массив**

```swift
let emptyArray: [String] = []
print(emptyArray[0]) // ❌ Index out of range
```

---

**Пример 3: Цикл с ошибкой**

```swift
let arr = [10, 20, 30]
for i in 0...arr.count { // ❌ arr.count = 3, допустимые индексы: 0...2
    print(arr[i])
}
```

---

**Пример 4: String и индексы**

```swift
let text = "Hi"
let index = text.index(text.startIndex, offsetBy: 3)
print(text[index]) // ❌ Index out of range
```

- Длина строки = 2 → offsetBy 3 выходит за пределы.
    

---

### Как исправить

#### 1️⃣ Проверять границы массива

```swift
if numbers.indices.contains(3) {
    print(numbers[3])
}
```

---

#### 2️⃣ Использовать безопасный доступ через [[first]], `last`, `dropFirst()`, `dropLast()`

```swift
print(numbers.first ?? 0) // ✅ безопасно
```

---

#### 3️⃣ Циклы через `indices` или [[for-in]]

```swift
for i in numbers.indices {
    print(numbers[i])
}

for number in numbers {
    print(number) // ✅ безопасно, без индексов
}
```

---

#### 4️⃣ Для строк использовать безопасные методы

```swift
if text.indices.contains(text.index(text.startIndex, offsetBy: 1)) {
    let char = text[text.index(text.startIndex, offsetBy: 1)]
    print(char)
}
```

---

### Резюме

- Ошибка возникает, когда **индекс выходит за пределы коллекции**.
    
- Исправляется через:
    
    1. Проверку индекса перед доступом.
        
    2. Использование безопасного доступа (`first`, `last`).
        
    3. Циклы через `indices` или `for-in`.
        
    4. Для String использовать безопасные индексы.
        
- Позволяет избежать **runtime crash** и безопасно работать с массивами и строками.
    

---
