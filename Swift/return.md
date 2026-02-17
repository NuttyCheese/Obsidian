**`return`** — оператор в [[Swift]], который **завершает выполнение функции, метода или замыкания** и **возвращает значение** (если функция не `Void`).  
Используется для **досрочного выхода из функции** или передачи результата вызвавшему коду.

---

## 🔹 Примеры кода

### 1. Функция без возвращаемого значения

```swift
func greet(name: String) {
    print("Hello, \(name)!")
    return // необязательно, можно просто выйти из функции
}

greet(name: "Alice")
```

---

### 2. Функция с возвращаемым значением

```swift
func sum(a: Int, b: Int) -> Int {
    return a + b
}

let result = sum(a: 3, b: 5) // 8
```

---

### 3. Досрочный выход из функции

```swift
func checkAge(_ age: Int) {
    guard age >= 18 else {
        print("Недостаточно лет")
        return // прерываем выполнение функции
    }
    print("Доступ разрешён")
}

checkAge(16)
checkAge(20)
```

---

### 4. Использование в замыканиях

```swift
let multiply: (Int, Int) -> Int = { a, b in
    return a * b
}

print(multiply(3, 4)) // 12
```

---

### 5. В циклах и [[switch]]

```swift
func findEven(numbers: [Int]) -> Int? {
    for num in numbers {
        if num % 2 == 0 {
            return num // возвращаем первое чётное число
        }
    }
    return nil // если не нашли
}

print(findEven(numbers: [1,3,5,6])) // 6
```
