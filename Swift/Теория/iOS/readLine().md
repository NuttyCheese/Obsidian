#swift #stdlib #io #console #input #command-line

---
## `readLine()` — Чтение строки из стандартного ввода
### Определение

**`readLine()`** — это функция из стандартной библиотеки [[Swift]], которая **считывает строку из стандартного ввода** (обычно с клавиатуры в консольном приложении). Она возвращает [[String]]`?` (опциональную строку), которая содержит введённые данные до символа новой строки (`\n`). Функция возвращает [[nil]], если достигнут конец файла (EOF) или ввод невозможен.

Это основной (и самый простой) способ получения пользовательского ввода в консольных программах на Swift.

---

### Синтаксис

```swift
func readLine(strippingNewline: Bool = true) -> String?
```

- **`strippingNewline`** — если `true` (по умолчанию), символ новой строки `\n` удаляется из результата. Если `false`, сохраняется.
- **Возвращает** — `String?` (опциональная строка) или `nil` при EOF.

---

### Базовые примеры

#### 1. **Простое чтение строки**

```swift
print("Enter your name: ", terminator: "")
if let name = readLine() {
    print("Hello, \(name)!")
} else {
    print("No input received")
}
```

#### 2. **Чтение без удаления новой строки**

```swift
let input = readLine(strippingNewline: false)
print(input as Any)  // Вывод: Optional("Hello\n")
```

#### 3. **Обработка нескольких строк**

```swift
print("Enter three lines:")
for i in 1...3 {
    if let line = readLine() {
        print("Line \(i): \(line)")
    }
}
```

---

### Чтение чисел

`readLine()` возвращает строку. Чтобы получить число, нужно преобразовать:

```swift
print("Enter your age: ", terminator: "")
if let input = readLine(), let age = Int(input) {
    print("You are \(age) years old")
} else {
    print("Invalid number")
}
```

#### Безопасное чтение чисел с повторным запросом

```swift
func readInt(prompt: String) -> Int {
    while true {
        print(prompt, terminator: "")
        guard let input = readLine(),
              let number = Int(input) else {
            print("Invalid input. Please enter a number.")
            continue
        }
        return number
    }
}

let age = readInt(prompt: "Enter your age: ")
print("Age: \(age)")
```

---

### Чтение нескольких значений в одной строке

```swift
print("Enter two numbers separated by space: ", terminator: "")
if let line = readLine() {
    let parts = line.split(separator: " ")
    if parts.count >= 2,
       let a = Int(parts[0]),
       let b = Int(parts[1]) {
        print("Sum: \(a + b)")
    }
}
```

---

### Обработка пустого ввода

```swift
print("Enter something (or press Enter to skip): ", terminator: "")
let input = readLine()

if let input, !input.isEmpty {
    print("You entered: \(input)")
} else {
    print("Nothing entered")
}
```

---

### Чтение до EOF (например, из файла или перенаправленного ввода)

```swift
print("Reading until EOF...")
while let line = readLine() {
    print("Read: \(line)")
}
print("EOF reached")
```

Это полезно при перенаправлении ввода:
```bash
swift myprogram.swift < input.txt
```

---

### Пример: интерактивная программа-калькулятор

```swift
func calculator() {
    print("Simple Calculator")
    print("Commands: +, -, *, /, exit")
    
    while true {
        print("\nEnter command: ", terminator: "")
        guard let command = readLine()?.lowercased() else { break }
        
        if command == "exit" {
            print("Goodbye!")
            break
        }
        
        print("Enter first number: ", terminator: "")
        guard let aStr = readLine(), let a = Double(aStr) else {
            print("Invalid number")
            continue
        }
        
        print("Enter second number: ", terminator: "")
        guard let bStr = readLine(), let b = Double(bStr) else {
            print("Invalid number")
            continue
        }
        
        let result: Double?
        switch command {
        case "+": result = a + b
        case "-": result = a - b
        case "*": result = a * b
        case "/": result = b != 0 ? a / b : nil
        default: result = nil
        }
        
        if let result = result {
            print("Result: \(result)")
        } else {
            print("Invalid command or division by zero")
        }
    }
}

calculator()
```

---

### Особенности и ограничения

| Особенность           | Описание                                                             |
| --------------------- | -------------------------------------------------------------------- |
| **Только консоль**    | `readLine()` работает только в консольных (command-line) приложениях |
| **Блокирующий вызов** | Ждёт ввода пользователя, пока не будет нажат Enter                   |
| **Кодировка**         | Использует системную кодировку (обычно UTF-8)                        |
| **Многопоточность**   | Не потокобезопасен (используйте один поток для ввода)                |
| **[[iOS]]/macOS GUI** | **Не работает** в приложениях с графическим интерфейсом              |

---

### Альтернативы для GUI-приложений

| Платформа   | Альтернатива                                                             |
| ----------- | ------------------------------------------------------------------------ |
| **iOS**     | [[UITextField]], [[UITextView]], [[UIAlertController]] с текстовым полем |
| **macOS**   | `NSTextField`, `NSTextView`                                              |
| **SwiftUI** | `TextField`, `TextEditor`                                                |

```swift
// SwiftUI пример
import SwiftUI

struct ContentView: View {
    @State private var userInput = ""
    
    var body: some View {
        TextField("Enter your name", text: $userInput)
            .textFieldStyle(RoundedBorderTextFieldStyle())
            .padding()
        
        Text("Hello, \(userInput)")
    }
}
```

---

### Лучшие практики

1.  **Всегда разворачивайте опциональный результат** — `readLine()` может вернуть `nil` (EOF).
2.  **Используйте `trimmingCharacters(in: .whitespacesAndNewlines)`** для удаления лишних пробелов.
3.  **Для чисел используйте безопасное преобразование** ([[Int]]`()`, [[Double]]`()`, возвращающие опционалы).
4.  **Не используйте `readLine()` в GUI-приложениях** — только в консольных.
5.  **Добавляйте понятные приглашения** (`print("Enter value: ", terminator: "")`).

```swift
// Хорошо: понятное приглашение и обработка пустого ввода
print("Enter city: ", terminator: "")
let city = readLine()?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
if !city.isEmpty {
    print("City: \(city)")
}
```

---

### Короткое правило

> **`readLine()`** — читает строку из консоли, возвращает `String?`.  
> Используется только в **command-line** приложениях.  
> Всегда проверяйте на `nil` и преобразуйте в числа через `Int()`/`Double()`.

---

### Итог

**`readLine()`** — это простой и удобный способ получить пользовательский ввод в консольных программах на Swift:

| Характеристика | Значение |
|---|---|
| **Тип возврата** | `String?` |
| **Удаление `\n`** | По умолчанию да (`strippingNewline: true`) |
| **Применение** | Только command-line приложения |
| **Блокировка** | Да (ждёт ввода) |
| **Возврат `nil`** | При EOF (Ctrl+D / Ctrl+Z) |

Понимание `readLine()` необходимо для создания интерактивных консольных утилит, скриптов и инструментов командной строки на Swift.