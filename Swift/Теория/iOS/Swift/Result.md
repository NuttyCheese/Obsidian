**`Result`** — это **[[enum]] с двумя кейсами**, который описывает результат операции:

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

- `Success` — тип значения в случае успеха
    
- `Failure` — тип ошибки, обязательно соответствует [[Error]]
    
- Позволяет безопасно **возвращать результат или ошибку** вместо [[Optional]] или кидания исключений
    

> Проще говоря: Result = «операция либо успешна с результатом, либо ошибка».

---

## 2. Основные термины

| Термин                 | Описание                                                          |
| ---------------------- | ----------------------------------------------------------------- |
| **Success**            | Значение, возвращаемое при успешной операции                      |
| **Failure**            | Ошибка, возвращаемая при неудаче (Error)                          |
| **Enum**               | Result — перечисление с кейсами `.success` и `.failure`           |
| **Pattern Matching**   | Использование [[switch]] или [[if case]] для обработки результата |
| **Throwing Functions** | Альтернатива Result — функции, которые кидают ошибки              |

---

## 3. Основной синтаксис

```swift
func divide(_ a: Int, by b: Int) -> Result<Int, Error> {
    if b == 0 {
        return .failure(NSError(domain: "DivideError", code: 1))
    } else {
        return .success(a / b)
    }
}

let result = divide(10, by: 2)
switch result {
case .success(let value):
    print("Result: \(value)")
case .failure(let error):
    print("Error: \(error.localizedDescription)")
}
```

- `Result<Int, Error>` → операция возвращает либо [[Int]], либо ошибку
    
- `switch` используется для **обработки кейсов**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
enum SimpleError: Error {
    case somethingWentWrong
}

func doTask(_ succeed: Bool) -> Result<String, SimpleError> {
    succeed ? .success("Task completed") : .failure(.somethingWentWrong)
}

let taskResult = doTask(true)

if case let .success(message) = taskResult {
    print(message) // Task completed
}
```

- Простейший пример с `if case` для извлечения success
    

---

### Пример 2. Использование switch

```swift
let failureResult: Result<String, SimpleError> = .failure(.somethingWentWrong)

switch failureResult {
case .success(let message):
    print(message)
case .failure(let error):
    print("Failed with error: \(error)")
}
```

- Обработка success и failure через `switch`
    

---

### Пример 3. [[map]] и [[flatMap]]

```swift
let result: Result<Int, SimpleError> = .success(10)
let doubled = result.map { $0 * 2 }
print(doubled) // success(20)
```

- `map` позволяет **трансформировать Success** без изменения Failure
    

---

### Пример 4. Throwing function с Result

```swift
func fetchData(from url: String) -> Result<Data, Error> {
    guard let data = url.data(using: .utf8) else {
        return .failure(NSError(domain: "URL", code: 404))
    }
    return .success(data)
}

let dataResult = fetchData(from: "Hello")
switch dataResult {
case .success(let data):
    print(String(data: data, encoding: .utf8)!) // Hello
case .failure(let error):
    print(error)
}
```

- Можно использовать для **синхронных операций, которые могут упасть**
    

---

### Пример 5. Обработка с `try?` и Result

```swift
enum FileError: Error {
    case notFound
}

func readFile(_ path: String) throws -> String {
    if path == "exists.txt" { return "File content" }
    else { throw FileError.notFound }
}

func safeRead(_ path: String) -> Result<String, Error> {
    do {
        let content = try readFile(path)
        return .success(content)
    } catch {
        return .failure(error)
    }
}

let fileResult = safeRead("exists.txt")
if case let .success(text) = fileResult {
    print(text) // File content
}
```

- Result может **оборачивать throwing функции**
    

---

## 5. Особенности Result

1. **Enum с кейсами success и failure**
    
2. Позволяет **явно возвращать ошибку или результат**
    
3. Совместим с **map, flatMap, [[try]]?**
    
4. Альтернатива Optional + [[throws]]
    
5. Удобен для **синхронных и асинхронных операций**
    

---

## 6. Итог

- **Result** = тип для безопасной работы с успехом и ошибкой
    
- Используется вместо `Optional` + `throw` для явного контроля ошибок
    
- Подходит для **синхронного кода, сетевых запросов, вычислений и обёртки throwing функций**
    
- Работает с **switch, if case, map, flatMap**
    

---
