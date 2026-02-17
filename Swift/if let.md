**`if let`** — это способ **безопасно извлечь значение из [[Optional]]**, выполняя блок кода **только если значение присутствует** (`!= nil`).

- Используется для Optional типов: `Int?`, `String?`, `[Type]?`
    
- Если Optional равен [[nil]] → блок кода **не выполняется**
    
- Позволяет работать с извлечённой переменной **как с обычным типом**, без `!`
    

> Проще говоря: `if let` = «если значение есть, извлечь его и использовать внутри блока».

---

## 2. Основные термины

| Термин               | Описание                                                      |
| -------------------- | ------------------------------------------------------------- |
| **Optional**         | Тип, который может быть `nil` (`Int?`, `String?`)             |
| **[[Unwrapping]]**   | Извлечение значения из Optional                               |
| **Optional Binding** | Присвоение значения Optional новой переменной через `if let`  |
| **Early Exit**       | Альтернатива [[guard let]] для работы с Optional внутри блока |
| **Scope**            | Извлечённая переменная доступна только внутри блока `if let`  |

---

## 3. Основной синтаксис

```swift
let name: String? = "Alice"

if let unwrappedName = name {
    print("Hello, \(unwrappedName)")
} else {
    print("Name is nil")
}
```

- `if let unwrappedName = name` → проверка на `nil`
    
- Внутри блока переменная `unwrappedName` **не Optional**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
let age: Int? = 25

if let age = age {
    print("Age is \(age)")
} else {
    print("Age is nil")
}
// Output: Age is 25
```

- Проверка Optional и извлечение значения
    

---

### Пример 2. Несколько Optional

```swift
let user: String? = "Alice"
let password: String? = "1234"

if let user = user, let password = password {
    print("Logging in \(user)")
} else {
    print("Missing credentials")
}
// Output: Logging in Alice
```

- Можно извлекать **несколько Optional одновременно**
    

---

### Пример 3. Приведение типов

```swift
let input: Any = "Hello"

if let text = input as? String {
    print("Text: \(text)")
}
// Output: Text: Hello
```

- Используем `if let` с `as?` для **безопасного приведения типов**
    

---

### Пример 4. Optional chaining + if let

```swift
class Pet {
    var name: String?
}

let myPet: Pet? = Pet()
myPet?.name = "Buddy"

if let petName = myPet?.name {
    print("Pet's name is \(petName)")
}
// Output: Pet's name is Buddy
```

- Позволяет **безопасно извлекать значения через Optional chaining**
    

---

### Пример 5. [[Dictionary]] + if let

```swift
let scores: [String: Int] = ["Alice": 10, "Bob": 15]

if let aliceScore = scores["Alice"] {
    print("Alice's score: \(aliceScore)")
}
// Output: Alice's score: 10
```

- При доступе через ключ словаря, возвращается **Optional**, который удобно извлекать через `if let`
    

---

## 5. Особенности `if let`

1. Используется для **безопасного извлечения Optional**
    
2. Извлечённая переменная доступна **только внутри блока**
    
3. Можно использовать с **несколькими Optional и условиями**
    
4. Часто применяется для **проверки значений из [[API]], словарей, Optional объектов**
    
5. Альтернатива **guard let**, но отличается областью видимости: `if let` → блок, [[guard let]] → весь остальной код
    

---

## 6. Итог

- **[[if let]]** = безопасное извлечение [[Optional]] с проверкой на [[nil]]
    
- Используется для **работы с Optional без [[Force unwrap]]**
    
- Позволяет уменьшить вероятность [[Runtime]] ошибок и улучшить читаемость кода
    

---
