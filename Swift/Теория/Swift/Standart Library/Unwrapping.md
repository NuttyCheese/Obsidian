## 1. Что такое `Unwrapping`

**Unwrapping** — это процесс получения **неопционального значения из [[Optional]]** (`?`).

- Optional = тип, который может хранить значение или [[nil]]
    
- Unwrapping нужен, чтобы **безопасно работать с этим значением**
    
- [[Swift]] предлагает **несколько способов извлечения значения**
    

> Проще говоря: Unwrapping = «достать реальное значение из Optional, если оно есть».

---

## 2. Основные термины

| Термин                | Описание                                              |
| ------------------------- | ----------------------------------------------------- |
| **Optional**          | Тип, который может быть `nil` (`Int?`, `String?`)     |
| **[[Force unwrap]]**  | Извлечение значения через `!`, риск падения, если nil |
| **Optional Binding**  | Безопасное извлечение через `if let` или `guard let`  |
| **Nil-Coalescing**    | `??` позволяет задать значение по умолчанию           |
| **Optional Chaining** | `?.` безопасно вызывает методы/свойства Optional      |

---

## 3. Основной синтаксис

```swift
var name: String? = "Alice"
if let unwrappedName = name {
    print(unwrappedName) // Alice
}
```

- [[if let]] = безопасный способ извлечь значение
    
- `unwrappedName` внутри блока — не Optional
    

---

## 4. Примеры от простого к сложному

### Пример 1. Force Unwrap

```swift
var number: Int? = 10
print(number!) // 10
```

- Используется `!` для извлечения значения
    
- **Опасно**, если `number = nil` → падение программы
    

---

### Пример 2. Optional Binding (if let)

```swift
var name: String? = "Bob"

if let unwrappedName = name {
    print("Hello, \(unwrappedName)")
} else {
    print("No name provided")
}
```

- Безопасный способ
    
- Можно использовать `else` для обработки `nil`
    

---

### Пример 3. Optional Binding ([[guard let]])

```swift
func greet(name: String?) {
    guard let unwrappedName = name else {
        print("No name provided")
        return
    }
    print("Hello, \(unwrappedName)")
}

greet(name: "Alice") // Hello, Alice
```

- `guard let` → удобен для **раннего выхода**
    
- Значение `unwrappedName` доступно **после guard**
    

---

### Пример 4. Nil-Coalescing (`??`)

```swift
var input: String? = nil
let output = input ?? "Default Value"
print(output) // Default Value
```

- Если Optional = nil → используется значение по умолчанию
    

---

### Пример 5. Optional Chaining

```swift
struct Person {
    var name: String?
    var address: String?
}

let person: Person? = Person(name: "Alice", address: nil)
print(person?.name)     // Optional("Alice")
print(person?.address)  // nil
```

- `?.` позволяет безопасно вызывать свойства/методы Optional
    
- Не вызывает краха при `nil`
    

---

### Пример 6. Combinations (Advanced)

```swift
struct User {
    var name: String?
    var age: Int?
}

let user: User? = User(name: "Tom", age: nil)

// Safe unwrapping with optional chaining + nil-coalescing
let userName = user?.name ?? "Unknown"
let userAge = user?.age ?? 0

print("\(userName), \(userAge)") // Tom, 0
```

- Совмещаем **optional chaining + nil-coalescing** для безопасного извлечения
    

---

## 5. Особенности Unwrapping

1. **Обязателен для работы с Optional**
    
2. Есть несколько подходов: `!`, `if let`, `guard let`, `??`, `?.`
    
3. Force unwrap опасен, может привести к crash
    
4. Optional binding позволяет **безопасно использовать значение** внутри блока
    
5. Nil-coalescing позволяет **подставить значение по умолчанию**
    

---

## 6. Итог

- **Unwrapping** = извлечение значения из [[Optional]]
    
- Используется для **безопасной работы с потенциально nil значениями**
    
- Основные способы:
    
    - **Force Unwrap (!)** – рискованно
        
    - **[[if let]] / [[guard let]]** – безопасно
        
    - **Nil-coalescing (??)** – значение по умолчанию
        
    - **Optional chaining (?.)** – безопасный доступ к свойствам и методам
        

---
