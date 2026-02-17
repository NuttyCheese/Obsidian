**`Error`** — это **протокол**, которому должен соответствовать тип, чтобы его можно было **выбрасывать ([[throws]]) и ловить (`catch`)** в [[Swift]].

- Используется для **обработки ошибок в функциях и методах**
    
- Любой тип ([[struct]], [[enum]], [[class]]), соответствующий протоколу `Error`, можно выбросить с `throw`
    
- В сочетании с `do-catch` позволяет писать безопасный код
    

> Проще говоря: Error = «тип ошибки, который можно бросить и поймать».

---

## 2. Основные термины

| Термин                    | Описание                                                   |
| ------------------------- | ---------------------------------------------------------- |
| **Error**                 | Протокол для создания типов ошибок                         |
| **throws**                | Ключевое слово для функции, которая может выбросить ошибку |
| **throw**                 | Выброс конкретной ошибки                                   |
| **[[do-catch]]**          | Блок для безопасного вызова функции с `throws`             |
| **[[try]] / try? / try!** | Вызов функции с обработкой ошибки или без неё              |

---

## 3. Основной синтаксис

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
    print("Caught error:", error)
}
```

- Enum `MyError` соответствует `Error`
    
- Ошибка выбрасывается через `throw` и ловится в `catch`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший enum с Error

```swift
enum NetworkError: Error {
    case offline
    case timeout
}

func fetchData() throws {
    throw NetworkError.offline
}

do {
    try fetchData()
} catch {
    print(error) // offline
}
```

- Enum с разными вариантами ошибок
    

---

### Пример 2. Struct как Error

```swift
struct ValidationError: Error {
    let field: String
    let message: String
}

func validateName(_ name: String) throws {
    if name.isEmpty {
        throw ValidationError(field: "name", message: "Name cannot be empty")
    }
}

do {
    try validateName("")
} catch let error as ValidationError {
    print("\(error.field): \(error.message)")
}
```

- Структуры тоже могут быть типом ошибки
    

---

### Пример 3. Error с [[Associated value]]

```swift
enum FileError: Error {
    case notFound(String)
    case permissionDenied(String)
}

func openFile(_ name: String) throws {
    throw FileError.notFound(name)
}

do {
    try openFile("data.txt")
} catch FileError.notFound(let fileName) {
    print("File not found:", fileName)
} catch {
    print("Other error")
}
```

- Associated values позволяют хранить **дополнительную информацию об ошибке**
    

---

### Пример 4. Error и Result type

```swift
enum NetworkError: Error { case offline, timeout }

func fetchData() -> Result<String, Error> {
    if Bool.random() {
        return .success("Data loaded")
    } else {
        return .failure(NetworkError.offline)
    }
}

let result = fetchData()
switch result {
case .success(let data): print(data)
case .failure(let error): print(error)
}
```

- Result type можно использовать как альтернативу `do-catch` для асинхронного кода
    

---

### Пример 5. [[async]]/[[await]] и Error

```swift
enum NetworkError: Error { case offline, timeout }

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
        print("Async error:", error)
    }
}
```

- Async функции тоже используют `throws` и `Error`
    

---

## 5. Особенности Error

1. **Протокол** — любой тип, который соответствует `Error`, можно выбросить
    
2. **Сочетается с throws / try / do-catch**
    
3. **Enum** — наиболее часто используемый способ, особенно с associated values
    
4. **Struct и class** — тоже могут быть Error
    
5. Позволяет писать **безопасный код и обрабатывать ошибки с контекстом**
    

---

## 6. Итог

- **Error** = протокол для создания типов ошибок
    
- Используется с **throws / try / do-catch / Result**
    
- Enum с case и associated values — основной способ моделирования ошибок
    
- Позволяет писать безопасный, читаемый и предсказуемый код
    

---
