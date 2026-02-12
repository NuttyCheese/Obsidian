## 1. Что такое `do-catch`

**`do-catch`** — блок кода, в котором можно вызывать функции, которые **могут выбросить ошибку ([[throws]])**, и безопасно её обрабатывать.

- `do` — блок, где выполняется код
    
- `catch` — блоки для отлова ошибок
    
- Позволяет отделить **ошибки от основной логики**
    

> Проще говоря: `do-catch` = «попробуй выполнить код и поймай ошибки, если они возникнут».

---

## 2. Основные термины

| Термин            | Описание                                               |
| ----------------- | ------------------------------------------------------ |
| `do`              | Блок, где выполняется потенциально опасный код         |
| [[try]]           | Используется для вызова функций с `throws`             |
| `catch`           | Блок для обработки ошибки                              |
| `catch ErrorType` | Можно ловить конкретный тип ошибки                     |
| `try?`            | Превращает результат в Optional, без падения программы |
| `try!`            | Принудительный вызов, ошибка приведёт к краху          |

---

## 3. Основной синтаксис

```swift
do {
    try riskyFunction()
} catch {
    print("Error:", error)
}
```

- Любая ошибка, выброшенная `riskyFunction`, попадёт в `catch`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
enum MyError: Error {
    case failed
}

func riskyFunction() throws {
    throw MyError.failed
}

do {
    try riskyFunction()
} catch {
    print("Caught an error:", error)
}
```

- Output: `"Caught an error: failed"`
    

---

### Пример 2. Несколько `catch` для разных ошибок

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
} catch {
    print("Other error")
}
```

- Позволяет **разделить обработку разных ошибок**
    

---

### Пример 3. try? и [[Optional]]

```swift
let result = try? fetchData()
// result = nil, если выброшена ошибка
```

- Не выбрасывает ошибку, превращает результат в Optional
    

---

### Пример 4. try! — принудительный вызов

```swift
try! riskyFunction() // если выбросит ошибку — падение приложения
```

- Используется только если вы **абсолютно уверены**, что ошибки не будет
    

---

### Пример 5. do-catch с [[async]]/[[await]]

```swift
func asyncFetch() async throws -> String {
    if Bool.random() {
        return "Data"
    } else {
        throw NetworkError.timeout
    }
}

Task {
    do {
        let data = try await asyncFetch()
        print(data)
    } catch {
        print("Error in async fetch:", error)
    }
}
```

- Для асинхронных функций используется **`try await`** внутри `do-catch`
    

---

### Пример 6. Rethrow + do-catch

```swift
func performTask(task: () throws -> Void) rethrows {
    try task()
}

do {
    try performTask {
        throw MyError.failed
    }
} catch {
    print("Caught rethrown error:", error)
}
```

- `rethrows` позволяет функции выбрасывать ошибку **только если [[closure]] выбросит**
    

---

## 5. Особенности do-catch

1. Используется с функциями `throws`
    
2. Позволяет ловить **конкретные ошибки**
    
3. Можно использовать `try?` и `try!` для удобства
    
4. Совместим с **async/await**
    
5. Позволяет безопасно разделять **логическую обработку** и **ошибки**
    

---

## 6. Итог

- **do-catch** = блок для безопасного вызова функций с `throws`
    
- `try` — вызов функции
    
- `catch` — обработка ошибки
    
- Можно ловить разные типы ошибок и комбинировать с `async/await`
    

---
