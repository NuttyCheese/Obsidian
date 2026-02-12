## 1. Что такое `throws`

**`throws`** — это ключевое слово, которое указывает, что **функция или метод может выбросить ошибку**.

- В [[Swift]] ошибки представляют собой типы, соответствующие протоколу **[[Error]]**.
    
- Функция с `throws` может завершиться либо **успешно** (return), либо **с ошибкой** (throw).
    
- Для вызова такой функции нужно использовать [[try]] (или `try?`, `try!`).
    

> Проще говоря: `throws` = «эта функция может не сработать и выдать ошибку, её нужно ловить».

---

## 2. Основные термины

| Термин       | Описание                                                           |
| ------------ | ------------------------------------------------------------------ |
| `throws`     | Функция может выбросить ошибку                                     |
| `throw`      | Ключевое слово для генерации ошибки внутри функции                 |
| `try`        | Используется при вызове функции с `throws`                         |
| `try?`       | Превращает результат в Optional, игнорируя ошибку (возвращает nil) |
| `try!`       | Принудительный вызов, ошибка приведёт к краху приложения           |
| [[do-catch]] | Блок для отлова и обработки ошибок                                 |

---

## 3. Создание функции с throws

```swift
enum MyError: Error {
    case somethingWentWrong
}

func riskyFunction() throws -> String {
    if Bool.random() {
        return "Success"
    } else {
        throw MyError.somethingWentWrong
    }
}
```

---

## 4. Вызов функции с throws

### Вариант 1. try + do-catch

```swift
do {
    let result = try riskyFunction()
    print("Result:", result)
} catch {
    print("Error:", error)
}
```

- Ошибки ловятся в `catch`
    
- Можно использовать разные catch для разных типов ошибок
    

```swift
do {
    let result = try riskyFunction()
} catch MyError.somethingWentWrong {
    print("Specific error")
} catch {
    print("Other error")
}
```

---

### Вариант 2. try? → [[Optional]]

```swift
let result = try? riskyFunction()
print(result) // Optional("Success") или nil
```

- Ошибка превращается в [[nil]]
    
- Не выбрасывает runtime exception
    

---

### Вариант 3. try! → принудительно

```swift
let result = try! riskyFunction() 
// Если функция выбросит ошибку → приложение упадёт
```

- Используется, когда вы **абсолютно уверены**, что ошибки не будет
    

---

## 5. Примеры от простого к сложному

### Пример 1. Простая функция с throws

```swift
enum FileError: Error {
    case fileNotFound
}

func readFile(name: String) throws -> String {
    guard name == "exists.txt" else {
        throw FileError.fileNotFound
    }
    return "File content"
}

do {
    let content = try readFile(name: "exists.txt")
    print(content)
} catch {
    print("Error reading file")
}
```

---

### Пример 2. Несколько ошибок

```swift
enum NetworkError: Error {
    case offline
    case timeout
}

func fetchData() throws {
    if Bool.random() {
        throw NetworkError.offline
    } else {
        throw NetworkError.timeout
    }
}

do {
    try fetchData()
} catch NetworkError.offline {
    print("Offline")
} catch NetworkError.timeout {
    print("Timeout")
}
```

---

### Пример 3. Использование try? для безопасного вызова

```swift
let result = try? fetchData() 
// result = nil, если выброшена ошибка
```

---

### Пример 4. throws в async функции

```swift
func asyncFetch() async throws -> String {
    if Bool.random() {
        return "Async data"
    } else {
        throw NetworkError.timeout
    }
}

Task {
    do {
        let data = try await asyncFetch()
        print(data)
    } catch {
        print("Error in async fetch")
    }
}
```

- Для `async throws` нужно использовать **`try await`**
    

---

### Пример 5. rethrow

```swift
func performTask(task: () throws -> Void) rethrows {
    try task()
}

do {
    try performTask {
        throw MyError.somethingWentWrong
    }
} catch {
    print("Caught error")
}
```

- `rethrows` позволяет функции **выбрасывать ошибку только если переданный [[closure]] выбросит**
    

---

## 6. Особенности throws

1. **Используется с Error-conforming типами**
    
2. **Обязателен `try` при вызове**
    
3. **Можно комбинировать с async** → `async throws`
    
4. **Можно безопасно превращать в Optional** → `try?`
    
5. **Можно принудительно вызвать** → `try!`
    
6. **Rethrow** позволяет передавать ошибки от closure
    

---

## 7. Итог

- `throws` = функция может выбросить ошибку
    
- Вызов через `try`, обработка через `do-catch`
    
- Для async используется `try await`
    
- Есть безопасные и принудительные варианты: `try?` и `try!`
    
- Rethrow позволяет безопасно передавать ошибки от closure
    

---
