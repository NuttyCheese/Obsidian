**`try`** — ключевое слово [[Swift]], которое используется для **вызова функции или метода, объявленного с [[throws]]**.

- Функция `throws` может выбросить ошибку, и её нужно **обрабатывать**
    
- `try` сообщает компилятору: «я осознаю, что эта функция может выбросить ошибку»
    
- Есть три варианта использования:
    
    1. **`try`** — обычный вызов в [[do-catch]]
        
    2. **`try?`** — возвращает Optional, если произошла ошибка → [[nil]]
        
    3. **`try!`** — принудительно, краш при ошибке
        

> Проще говоря: `try` = «попробовать выполнить функцию, которая может выбросить ошибку».

---

## 2. Основные термины

| Термин       | Описание                                             |
| ------------ | ---------------------------------------------------- |
| **throws**   | Функция может выбросить ошибку                       |
| **throw**    | Бросить ошибку                                       |
| **try**      | Вызов функции с обработкой ошибки                    |
| **try?**     | Преобразует результат в [[Optional]], nil при ошибке |
| **try!**     | Принудительный вызов, падение приложения при ошибке  |
| **do-catch** | Блок для перехвата ошибок                            |

---

## 3. Основной синтаксис

```swift
enum MyError: Error {
    case failed
}

func riskyFunction() throws -> String {
    throw MyError.failed
}

do {
    let result = try riskyFunction()
} catch {
    print("Caught error:", error)
}
```

- `riskyFunction` объявлена с `throws`
    
- Вызов через `try` обязательно внутри `do-catch`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Базовое использование `try`

```swift
enum NetworkError: Error {
    case offline
}

func fetchData() throws -> String {
    throw NetworkError.offline
}

do {
    let data = try fetchData()
} catch {
    print("Error:", error)
}
```

- Обычное использование `try` с `do-catch`
    

---

### Пример 2. try? → Optional

```swift
func risky() throws -> Int {
    if Bool.random() { return 10 }
    else { throw NetworkError.offline }
}

let value = try? risky()
print(value) // Optional(10) или nil
```

- Если функция выбросила ошибку → результат `nil`
    

---

### Пример 3. try! → принудительный вызов

```swift
func alwaysSuccess() throws -> String {
    return "Success"
}

let result = try! alwaysSuccess()
print(result) // Success
```

- Используется, когда точно известно, что ошибки не будет
    
- ❌ Если ошибка произойдёт → краш приложения
    

---

### Пример 4. Использование try в chained вызовах

```swift
enum FileError: Error {
    case notFound
}

func readFile() throws -> String { throw FileError.notFound }
func parseFile(_ content: String) throws -> Int { return content.count }

do {
    let count = try parseFile(try readFile())
} catch {
    print(error) // notFound
}
```

- Можно вызывать несколько `throws` функций внутри одной строки
    

---

### Пример 5. [[async]]/[[await]] + try

```swift
enum NetworkError: Error { case timeout }

func asyncFetch() async throws -> String {
    if Bool.random() { return "Data" }
    else { throw NetworkError.timeout }
}

Task {
    do {
        let data = try await asyncFetch()
        print(data)
    } catch {
        print("Async error:", error)
    }
}
```

- Асинхронные функции, которые могут выбросить ошибку, тоже используют `try`
    

---

## 5. Особенности try

1. Используется **только с функциями, которые имеют `throws`**
    
2. Три варианта: `try`, `try?`, `try!`
    
3. Позволяет безопасно обрабатывать ошибки с `do-catch`
    
4. Может использоваться в **асинхронных функциях** с `await`
    
5. Необязательно оборачивать в `do-catch` при `try?` или `try!`
    

---

## 6. Итог

- **try** = попытка вызвать функцию, которая может выбросить ошибку
    
- Позволяет **обрабатывать ошибки безопасно или преобразовать результат в Optional**
    
- Ключевой инструмент для **error handling** в Swift
    

---
