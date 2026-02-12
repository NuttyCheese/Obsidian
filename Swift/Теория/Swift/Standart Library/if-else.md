#standart_library #Swift 
## 📘 Определение

**`if-else`** — оператор ветвления в [[Swift]], который позволяет **выполнять разные блоки кода в зависимости от условия**.  
Используется для принятия решений на основе логических выражений ([[Swift/Теория/Swift/Standart Library/Bool]]).

---

## 🔹 Примеры кода

### 1. Простейший `if`

```swift
let number = 5

if number > 0 {
    print("Число положительное")
}
```

---

### 2. `if-else`

```swift
let number = -3

if number > 0 {
    print("Число положительное")
} else {
    print("Число не положительное")
}
```

---

### 3. `if-else if-else`

```swift
let number = 0

if number > 0 {
    print("Число положительное")
} else if number < 0 {
    print("Число отрицательное")
} else {
    print("Число равно нулю")
}
```

---

### 4. Использование с логическими операторами

```swift
let age = 20
let hasPermission = true

if age >= 18 && hasPermission {
    print("Доступ разрешен")
} else {
    print("Доступ запрещен")
}
```

---

### 5. Вложенные `if-else`

```swift
let a = 10
let b = 5

if a > 0 {
    if b > 0 {
        print("Оба числа положительные")
    } else {
        print("Первое положительное, второе нет")
    }
} else {
    print("Первое число не положительное")
}
```

---

### 6. Использование с опционалом

```swift
let name: String? = "Alice"

if let unwrappedName = name {
    print("Имя пользователя:", unwrappedName)
} else {
    print("Имя не задано")
}
```
