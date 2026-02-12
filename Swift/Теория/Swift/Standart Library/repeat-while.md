#standart_library #Swift 
## 📘 Определение

**`repeat-while`** — цикл в [[Swift]], который выполняет блок кода **как минимум один раз** и повторяет его **пока условие истинно**.  
Отличие от `while` в том, что проверка условия происходит **после выполнения тела цикла**.

---

## 🔹 Примеры кода

### 1. Простейший `repeat-while`

```swift
var count = 1

repeat {
    print("count = \(count)")
    count += 1
} while count <= 5
```

---

### 2. Использование с массивом

```swift
let numbers = [1, 2, 3]
var index = 0

repeat {
    print("Число:", numbers[index])
    index += 1
} while index < numbers.count
```

---

### 3. Прерывание цикла с [[break]]

```swift
var number = 0

repeat {
    number += 1
    if number == 3 {
        break // досрочно выходим
    }
    print("number =", number)
} while number < 10
```

---

### 4. С `continue` для пропуска итерации

```swift
var i = 0

repeat {
    i += 1
    if i % 2 == 0 {
        continue // пропускаем чётные числа
    }
    print("Нечётное число:", i)
} while i < 5
```

---

### 5. Валидация ввода пользователя

```swift
var input: String?

repeat {
    print("Введите текст:")
    input = readLine()
} while input == nil || input!.isEmpty

print("Вы ввели:", input!)
```
