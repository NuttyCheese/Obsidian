## 1. Что такое `Optional`

**`Optional`** — это **тип-контейнер**, который может содержать:

1. Значение определённого типа (`T`)
    
2. Или **[[nil]]**, если значения нет
    

- Обозначается через `?` после типа (`Int?`, `String?`)
    
- Позволяет **безопасно работать с отсутствующими данными**
    

> Проще говоря: Optional = «контейнер, который либо хранит значение, либо пустой (nil)».

---

## 2. Основные термины

| Термин                    | Описание                                                 |
| ------------------------- | -------------------------------------------------------- |
| **Optional**              | Тип данных, который может содержать значение или nil     |
| **nil**                   | Отсутствие значения в Optional                           |
| **[[Unwrapping]]**        | Извлечение значения из Optional                          |
| **[[Force unwrap]]**      | Принудительное извлечение через `!`                      |
| **Optional binding**      | Безопасное извлечение через [[if let]] или [[guard let]] |
| **Optional chaining**     | Доступ к свойствам или методам через Optional (`?.`)     |
| **Nil coalescing (`??`)** | Установка значения по умолчанию, если Optional равен nil |

---

## 3. Основной синтаксис

```swift
var name: String? = nil   // Optional без значения
name = "Alice"            // Теперь содержит значение
```

- Без `?` присвоение `nil` невозможно
    
- Optional может быть **[[generic]]**, например `Optional<Int>`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший Optional

```swift
var number: Int? = nil
print(number) // nil
number = 42
print(number) // Optional(42)
```

- Optional может быть пустым (`nil`) или содержать значение
    

---

### Пример 2. Force unwrap

```swift
let name: String? = "Bob"
print(name!) // Bob
// ❌ Если name = nil, будет crash
```

- `!` извлекает значение из Optional
    
- Нужно быть уверенным, что Optional не [[nil]]
    

---

### Пример 3. Optional binding с if let

```swift
let name: String? = "Alice"
if let unwrappedName = name {
    print("Hello, \(unwrappedName)")
} else {
    print("Name is nil")
}
```

- Безопасная распаковка Optional
    
- Избегает крашей при `nil`
    

---

### Пример 4. Guard let

```swift
func greet(_ name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    print("Hello, \(name)")
}

greet(nil)   // No name provided
greet("Eve") // Hello, Eve
```

- Guard позволяет **ранний выход**, если Optional пустой
    

---

### Пример 5. Nil coalescing (`??`)

```swift
let name: String? = nil
let displayName = name ?? "Unknown"
print(displayName) // Unknown
```

- Если Optional равен nil, используется значение по умолчанию
    

---

### Пример 6. Optional chaining

```swift
struct Person {
    var pet: Pet?
}

struct Pet {
    var name: String
}

let person = Person(pet: Pet(name: "Fluffy"))
print(person.pet?.name) // Optional("Fluffy")

let person2 = Person(pet: nil)
print(person2.pet?.name) // nil
```

- Позволяет безопасно обращаться к свойствам Optional
    

---

### Пример 7. Преобразование Optional к Non-Optional

```swift
let optionalNumber: Int? = 5
let number = optionalNumber ?? 0 // Если nil, присвоится 0
```

- Очень часто используется для **предотвращения nil**
    

---

## 5. Особенности Optional

1. **Тип-контейнер**, хранит значение или nil
    
2. **Generic** — `Optional<T>`
    
3. Требует распаковки для использования значения (`!`, `if let`, `guard let`)
    
4. Безопасные методы: **optional binding, optional chaining, nil coalescing**
    
5. Основной инструмент для **безопасного и читаемого кода**
    

---

## 6. Итог

- **Optional** = контейнер, который может хранить значение или быть пустым (`nil`)
    
- Используется для работы с **неопределёнными данными и безопасным кодом**
    
- Поддерживает **распаковку, optional chaining и nil coalescing**
    

---
Внутри Optional выглядит примерно так:
```swift

enum Optional<Wrapped> {     
	case none       // соответствует nil     
	case some(Wrapped)  // содержит значение типа Wrapped }`

```