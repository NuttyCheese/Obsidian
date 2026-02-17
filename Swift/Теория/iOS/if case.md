**`if case`** — это конструкция, которая позволяет **сравнивать значение с паттерном и извлекать [[Associated value]]**.

- Часто используется с **enum**, Optional и другими структурами, поддерживающими pattern matching
    
- Позволяет **сократить вложенные [[switch]] или [[if let]] конструкции**
    
- Можно комбинировать с **[[let]]** для извлечения значений
    

> Проще говоря: `if case` = «если значение совпадает с паттерном, то выполнить блок».

---

## 2. Основные термины

| Термин               | Описание                                                                |
| -------------------- | ----------------------------------------------------------------------- |
| **Pattern Matching** | Сравнение значения с паттерном, извлечение associated values            |
| **[[enum]]**         | Перечисление с кейсами, которые могут иметь associated values           |
| **Associated Value** | Дополнительное значение, хранящееся в кейсе enum                        |
| **Optional Pattern** | Проверка [[Optional]] через `.some(let value)` или `if case let value?` |
| **Let binding**      | Извлечение значения через `let` внутри паттерна                         |

---

## 3. Основной синтаксис

```swift
enum Result {
    case success(String)
    case failure(Error)
}

let result = Result.success("Data loaded")

if case .success(let message) = result {
    print("Success: \(message)")
}
```

- `if case .success(let message) = result` → проверяем, что `result` соответствует кейсу `success` и извлекаем `message`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Enum с associated value

```swift
enum Status {
    case online
    case offline
    case message(String)
}

let currentStatus = Status.message("Hello!")

if case .message(let text) = currentStatus {
    print("Received message: \(text)")
}
// Output: Received message: Hello!
```

- Извлекаем associated value из enum кейса
    

---

### Пример 2. Optional с `if case let`

```swift
let number: Int? = 42

if case let value? = number {
    print("Number is \(value)")
}
// Output: Number is 42
```

- Альтернатива `if let value = number`
    
- Работает с **Optional Pattern** (`value?`)
    

---

### Пример 3. Несколько паттернов через `,`

```swift
enum Event {
    case click(x: Int, y: Int)
    case swipe(direction: String)
}

let event = Event.click(x: 10, y: 20)

if case .click(let x, let y) = event, x > 5 {
    print("Clicked at \(x), \(y)")
}
// Output: Clicked at 10, 20
```

- Можно добавлять **дополнительные условия** через `,`
    

---

### Пример 4. Паттерн с диапазоном

```swift
let score = 85

if case 80...100 = score {
    print("Excellent score!")
}
// Output: Excellent score!
```

- Можно использовать **диапазоны** или другие паттерны, не только enum
    

---

### Пример 5. Паттерн с switch и if case

```swift
enum Direction {
    case north, south, east, west
}

let dir: Direction? = .north

if case .north? = dir {
    print("Heading north!")
}
// Output: Heading north!
```

- С Optional enum можно использовать `?` в паттерне
    

---

## 5. Особенности `if case`

1. **Работает с паттернами**, не только с конкретными значениями
    
2. Позволяет **извлекать associated values** из enum и Optional
    
3. Можно использовать **несколько условий через `,`**
    
4. Подходит для **сокращения switch/if-let вложенности**
    
5. Альтернатива `switch` для **одного кейса или проверки**
    

---

## 6. Итог

- **`if case`** = проверка значения на соответствие паттерну
    
- Особенно полезно с **enum, Optional и associated values**
    
- Позволяет **извлекать данные и добавлять условия** в компактной форме
    

---
