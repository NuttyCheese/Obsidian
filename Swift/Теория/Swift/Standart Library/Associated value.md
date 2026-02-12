## 1. Что такое Associated Value

**Associated Value** — это **данные, которые можно прикрепить к конкретному case enum**.

- Каждое перечисление ([[enum]]) может иметь **разные типы значений для разных кейсов**
    
- Это делает `enum` очень мощным для **моделирования данных с вариантами и параметрами**
    
- Отличие от raw value: raw value фиксирован для каждого case, а associated value **может быть разным при каждом создании экземпляра**
    

> Проще говоря: Associated Value = «дополнительные данные, которые можно хранить вместе с кейсом enum».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Enum case**|Конкретный вариант перечисления|
|**Associated Value**|Значение, привязанное к конкретному кейсу enum|
|**Switch-case**|Структура для обработки enum и извлечения associated values|
|**Tuple**|Часто используется для хранения нескольких associated values|
|**Pattern matching**|Распаковка associated values через `case let` или `case var`|

---

## 3. Основной синтаксис

```swift
enum Result {
    case success(value: Int)
    case failure(error: String)
}

let result: Result = .success(value: 42)
```

- `success` хранит [[Int]], `failure` хранит [[String]]
    
- При каждом создании экземпляра можно передать **разные значения**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Enum с одним associated value

```swift
enum NetworkResult {
    case success(Int)
    case failure(String)
}

let response = NetworkResult.success(200)

switch response {
case .success(let code):
    print("Success with code \(code)")
case .failure(let message):
    print("Failed with message: \(message)")
}
```

- Распаковка значений через `let`
    

---

### Пример 2. Enum с несколькими associated values

```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}

let productBarcode = Barcode.upc(8, 85909, 51226, 3)

switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem)-\(manufacturer)-\(product)-\(check)")
case .qrCode(let code):
    print("QR: \(code)")
}
```

- Можно хранить несколько значений через **tuple**
    

---

### Пример 3. Использование с [[Optional]] и [[if case]]

```swift
enum LoginStatus {
    case success(userID: String)
    case failure(error: String)
}

let status: LoginStatus? = .success(userID: "abc123")

if case .success(let userID)? = status {
    print("Logged in as \(userID)")
}
```

- Позволяет компактно проверять кейс без полного [[switch]]
    

---

### Пример 4. Enum с ассоциированными типами разных структур

```swift
struct User {
    let name: String
    let age: Int
}

struct Admin {
    let name: String
    let permissions: [String]
}

enum Person {
    case user(User)
    case admin(Admin)
}

let person: Person = .admin(Admin(name: "Alice", permissions: ["edit", "delete"]))

switch person {
case .user(let u):
    print("User: \(u.name)")
case .admin(let a):
    print("Admin: \(a.name), permissions: \(a.permissions)")
}
```

- Associated Value может быть **структурой, классом, кортежем**
    

---

### Пример 5. Enum + функции внутри associated values

```swift
enum Operation {
    case add(Int, Int)
    case multiply(Int, Int)
    
    func perform() -> Int {
        switch self {
        case .add(let a, let b): return a + b
        case .multiply(let a, let b): return a * b
        }
    }
}

let op: Operation = .add(3, 5)
print(op.perform()) // 8
```

- Associated Value позволяет **хранить параметры для функции или действия**
    

---

## 5. Особенности Associated Value

1. **Каждый case может хранить свой тип данных**
    
2. Значения задаются при создании экземпляра enum
    
3. Используется **switch-case или if-case** для извлечения данных
    
4. Можно хранить **кортежи, структуры, классы, функции, опционалы**
    
5. Отличие от raw value: raw value фиксирован для case, associated value динамический
    

---

## 6. Итог

- **Associated Value** = дополнительные данные, привязанные к case enum
    
- Позволяет создавать **гибкие, типобезопасные модели**
    
- Широко используется для **результатов, ошибок, действий с параметрами**
    

---
