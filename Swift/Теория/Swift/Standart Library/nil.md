## 1. Что такое `nil`

**`nil`** — это **специальное значение**, которое обозначает **отсутствие значения** у переменной или константы.

- Работает **только с [[Optional]] типами** (`?`)
    
- Позволяет безопасно обрабатывать ситуации, когда значение может отсутствовать
    
- Нельзя присвоить `nil` переменной без Optional
    

> Проще говоря: `nil` = «значение отсутствует».

---

## 2. Основные термины

| Термин                    | Описание                                                    |
| ------------------------- | ----------------------------------------------------------- |
| **Optional**              | Тип, который может содержать значение или `nil` (`String?`) |
| **[[Unwrapping]]**        | Извлечение значения из Optional                             |
| **Optional binding**      | Безопасная распаковка через [[if let]] или [[guard let]]    |
| **[[Force unwrap]]**      | Принудительная распаковка через `!`                         |
| **Nil coalescing (`??`)** | Установка значения по умолчанию, если Optional равен `nil`  |
| **Optional chaining**     | Доступ к свойствам или методам через Optional (`?.`)        |

---

## 3. Основной синтаксис

```swift
var name: String? = nil  // Optional без значения
name = "Alice"           // Теперь содержит значение
```

- Переменная типа `String?` может быть `nil` или содержать строку
    
- Прямое присвоение `nil` обычной переменной без `?` невозможно
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший Optional

```swift
var number: Int? = nil
print(number) // nil
number = 10
print(number) // Optional(10)
```

- Optional может быть пустым (`nil`) или содержать значение
    

---

### Пример 2. Force unwrap

```swift
var name: String? = "Alice"
print(name!) // Alice
```

- `!` принудительно извлекает значение
    
- ❌ Если `nil` — будет [[runtime crash]]
    

---

### Пример 3. Optional binding с if let

```swift
var name: String? = "Bob"
if let unwrappedName = name {
    print("Hello, \(unwrappedName)")
} else {
    print("Name is nil")
}
```

- Безопасная распаковка Optional
    
- Не вызывает краш, если `nil`
    

---

### Пример 4. Guard let

```swift
func greet(_ name: String?) {
    guard let name = name else {
        print("Name is nil")
        return
    }
    print("Hello, \(name)")
}

greet(nil)   // Name is nil
greet("Eve") // Hello, Eve
```

- Guard позволяет **ранний выход** из функции, если Optional равен `nil`
    

---

### Пример 5. Nil coalescing (`??`)

```swift
var name: String? = nil
let displayName = name ?? "Unknown"
print(displayName) // Unknown
```

- Если Optional `nil`, используется значение по умолчанию
    

---

### Пример 6. Optional chaining

```swift
struct Person {
    var pet: Pet?
}

struct Pet {
    var name: String
}

let person = Person(pet: nil)
print(person.pet?.name) // nil
```

- Позволяет безопасно обращаться к свойствам Optional, если они могут быть `nil`
    

---

## 5. Особенности `nil`

1. **Работает только с Optional** — обычная переменная не может быть nil
    
2. **Optional** = контейнер, который может быть пустым
    
3. Безопасные способы работы: **if let, guard let, optional chaining, nil coalescing**
    
4. Force unwrap `!` может привести к крашу
    
5. Используется для **моделирования отсутствующих значений и безопасного кода**
    

---

## 6. Итог

- `nil` = «значение отсутствует»
    
- Всегда связан с **Optional** (`?`)
    
- Безопасная работа через **optional binding, optional chaining и nil coalescing**
    
- Используется для **избежания крашей и обработки неопределённых данных**
    

---
