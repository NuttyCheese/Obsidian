## 1. Что такое `defer`

**`defer`** — это ключевое слово [[Swift]], которое позволяет **отложить выполнение блока кода до выхода из текущего scope** (функции, метода, цикла, условия).

- Всегда выполняется **в конце scope**, даже если возникнет ошибка или [[return]]
    
- Можно использовать **несколько defer блоков**, они выполняются в **обратном порядке (LIFO)**
    

> Проще говоря: `defer` = «выполнить этот код в конце текущего блока независимо от того, как он закончится».

---

## 2. Основные термины

| Термин                        | Описание                                                        |
| ----------------------------- | --------------------------------------------------------------- |
| **Scope**                     | Область видимости: функция, метод, цикл, условие                |
| **LIFO (Last In, First Out)** | Последний добавленный `defer` выполняется первым                |
| **Resource cleanup**          | Освобождение ресурсов, файлов, соединений, временных объектов   |
| **Error handling**            | `defer` выполняется даже если выброшено исключение ([[throws]]) |

---

## 3. Основной синтаксис

```swift
func example() {
    defer {
        print("Этот блок выполнится последним")
    }
    print("Этот блок выполнится первым")
}

example()
// Output:
// Этот блок выполнится первым
// Этот блок выполнится последним
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
func greet() {
    defer { print("Goodbye") }
    print("Hello")
}

greet()
// Output:
// Hello
// Goodbye
```

- `defer` выполнится после основного кода функции
    

---

### Пример 2. Несколько defer блоков

```swift
func multipleDefers() {
    defer { print("First defer") }
    defer { print("Second defer") }
    print("Function body")
}

multipleDefers()
// Output:
// Function body
// Second defer
// First defer
```

- **LIFO порядок** выполнения
    

---

### Пример 3. Defer + return

```swift
func checkNumber(_ number: Int) -> Bool {
    defer { print("Function is ending") }
    if number > 0 { return true }
    return false
}

checkNumber(5)
// Output:
// Function is ending
// true
```

- `defer` выполняется **даже при раннем return**
    

---

### Пример 4. Defer для очистки ресурсов

```swift
func readFile() {
    let file = "data.txt"
    print("Opening file \(file)")
    
    defer { print("Closing file \(file)") }
    
    print("Processing file \(file)")
}

readFile()
// Output:
// Opening file data.txt
// Processing file data.txt
// Closing file data.txt
```

- Используется для **освобождения ресурсов**, аналогично `finally` в других языках
    

---

### Пример 5. Defer с throw

```swift
enum MyError: Error { case error }

func riskyOperation() throws {
    defer { print("Cleanup before leaving function") }
    print("Doing risky stuff")
    throw MyError.error
}

do {
    try riskyOperation()
} catch {
    print("Caught error")
}

// Output:
// Doing risky stuff
// Cleanup before leaving function
// Caught error
```

- `defer` выполняется **даже при выбрасывании ошибки**
    

---

## 5. Особенности defer

1. **Выполняется в конце scope**
    
2. **Несколько defer** → выполняются в **обратном порядке**
    
3. Может использоваться для **очистки ресурсов, временных объектов, файлов, соединений**
    
4. Работает с **return, throw, break, continue** — всегда выполнится
    
5. Идеален для **управления ресурсами в функциях и методах**
    

---

## 6. Итог

- **defer** = механизм отложенного выполнения кода в конце scope
    
- Позволяет безопасно **очищать ресурсы и выполнять завершающие действия**
    
- Всегда выполняется, даже при **return или throw**
    
- Несколько defer выполняются **в обратном порядке добавления**
    

---
