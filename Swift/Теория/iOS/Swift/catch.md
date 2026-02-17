`catch` — часть конструкции обработки ошибок в [[Swift]].  
Используется вместе с `do` и `try` для перехвата и обработки ошибок, выбрасываемых методами или функциями.  
Позволяет реагировать на разные типы ошибок и предотвращает аварийное завершение программы.

---

## 🔹 Примеры кода

### 1. Базовый пример `do–try–catch`

```swift
enum FileError: Error {
    case notFound
}

func loadFile() throws {
    throw FileError.notFound
}

do {
    try loadFile()
} catch {
    print("Ошибка: \(error)")
}
```

---

### 2. Несколько блоков `catch` для разных ошибок

```swift
enum LoginError: Error {
    case wrongPassword
    case userNotFound
}

func login() throws {
    throw LoginError.wrongPassword
}

do {
    try login()
} catch LoginError.wrongPassword {
    print("Неверный пароль")
} catch LoginError.userNotFound {
    print("Пользователь не найден")
}
```

---

### 3. `catch` с привязкой значения

```swift
enum APIError: Error {
    case server(code: Int)
}

func request() throws {
    throw APIError.server(code: 500)
}

do {
    try request()
} catch APIError.server(let code) {
    print("Ошибка сервера: \(code)")
}
```

---

### 4. Универсальный `catch` после точечных

```swift
do {
    try request()
} catch APIError.server {
    print("Серверная ошибка")
} catch {
    print("Другая ошибка: \(error)")
}
```

---

### 5. Преобразование ошибки через `try?` (без throw, catch не обязателен)

```swift
func parse(_ text: String) throws -> Int {
    guard let number = Int(text) else { throw NSError() }
    return number
}

let result = try? parse("abc") // nil вместо ошибки
```

---

### 6. Ловим ошибку в асинхронном коде ([[async]]/[[await]])

```swift
enum NetworkError: Error { case timeout }

func loadData() async throws -> String {
    throw NetworkError.timeout
}

Task {
    do {
        let value = try await loadData()
        print(value)
    } catch {
        print("Ошибка загрузки: \(error)")
    }
}
```

---
