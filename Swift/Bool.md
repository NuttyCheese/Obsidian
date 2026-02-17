## 📘 Определение

`Bool` — логический тип данных в [[Swift]].  
Хранит два значения: `true` или `false`.  
Используется в условиях, проверках, ветвлениях, логических операциях и управлении потоком выполнения.

---

## 🔹 Примеры кода

### 1. Объявление переменной `Bool`

```swift
let isOnline: Bool = true
let isGuest = false
```

---

### 2. Использование в `if`

```swift
let isLoggedIn = false

if isLoggedIn {
    print("Добро пожаловать!")
} else {
    print("Пользователь не авторизован")
}
```

---

### 3. Логические операции (`&&`, `||`, `!`)

```swift
let a = true
let b = false

let andResult = a && b      // false
let orResult = a || b       // true
let notResult = !a          // false
```

---

### 4. Bool в тернарном операторе

```swift
let isDarkMode = true
let theme = isDarkMode ? "Dark" : "Light"

print(theme) // Dark
```

---

### 5. Bool как результат сравнения

```swift
let age = 20
let canBuy = age >= 18

if canBuy {
    print("Разрешено")
}
```

---

### 6. Создание свойства и наблюдение за изменением

```swift
struct Settings {
    var isEnabled: Bool {
        didSet {
            print("isEnabled изменён на \(isEnabled)")
        }
    }
}

var settings = Settings(isEnabled: false)
settings.isEnabled = true
```

---

### 7. Расширение Bool (например, функция toggle)

```swift
extension Bool {
    mutating func invert() {
        self = !self
    }
}

var flag = false
flag.invert() // true
```

---
