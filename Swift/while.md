**`while`** — оператор цикла в [[Swift]], который выполняет блок кода **пока условие истинно**.  
Используется, когда количество итераций заранее неизвестно.  
Относится к **Swift → Управление потоком**.

---

## 🔹 Примеры кода

### 1. Простейший `while`

```swift
var i = 0

while i < 5 {
    print("i = \(i)")
    i += 1
}
```

---

### 2. Бесконечный цикл с [[break]]

```swift
var count = 0

while true {
    count += 1
    if count == 3 {
        break // прерываем цикл
    }
}
print("count =", count) // 3
```

---

### 3. Использование [[continue]]

```swift
var number = 0

while number < 5 {
    number += 1
    if number % 2 == 0 {
        continue // пропускаем четные числа
    }
    print("Нечетное число:", number)
}
```

---

### 4. Чтение данных до конца

```swift
let inputs = ["one", "two", "stop", "three"]
var index = 0

while index < inputs.count {
    let value = inputs[index]
    if value == "stop" { break }
    print(value)
    index += 1
}
```

---

### 5. Использование с опционалом

```swift
var optionalNumber: Int? = 3

while let number = optionalNumber, number > 0 {
    print("Number =", number)
    optionalNumber = number - 1
}
```
