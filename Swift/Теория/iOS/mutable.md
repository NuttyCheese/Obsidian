**mutable** в [[Swift]] — это ключевое слово, которое используется для **разрешения изменения** значения свойства, даже если само свойство объявлено как `let` (константа).

Это один из самых важных и одновременно самых спорных механизмов в языке, особенно в контексте **Swift 6 strict concurrency** и **[[Value Type]]** ([[struct]], [[enum]]).

### Основные варианты использования `mutable` (2026 актуальные)

1. **mutable var в struct / enum**  
   Самый частый и основной случай — позволяет менять свойство внутри метода, даже если экземпляр объявлен как `let`.

```swift
struct User {
    var name: String
    var age: Int
    
    // mutable позволяет менять свойства внутри метода
    mutating func birthday() {
        age += 1
        name += " 🎉"  // тоже меняется
    }
}

var user = User(name: "Alex", age: 30)
user.birthday()           // OK

let constantUser = User(name: "Bob", age: 25)
// constantUser.birthday()  // Ошибка компиляции: Cannot use mutating member on immutable value
```

**Почему это важно**:  
Без `mutating` методы, изменяющие свойства структуры, **нельзя** вызывать на `let`-экземплярах.  
Это ключевое правило value semantics в Swift.

2. **mutable в протоколах** (mutating [[func]] / mutating [[var]])

```swift
protocol Updatable {
    mutating func updateName(_ newName: String)
}

struct Person: Updatable {
    var name: String
    
    mutating func updateName(_ newName: String) {
        name = newName
    }
}

var person = Person(name: "Alice")
person.updateName("Bob")  // OK

// let person = Person(name: "Charlie")
// person.updateName("David") // Ошибка: mutating func on immutable value
```

3. **mutable в extension** (часто забывают)

```swift
extension Array {
    mutating func appendIfNotEmpty(_ element: Element) {
        if !isEmpty {
            append(element)
        }
    }
}

var numbers = [1, 2, 3]
numbers.appendIfNotEmpty(4)  // OK

let constantArray = [5, 6]
// constantArray.appendIfNotEmpty(7) // Ошибка
```

### mutable vs non-mutating (ключевые отличия 2026)

| Характеристика                        | mutating func / [[var]]                     | non-mutating func / var  |
| ------------------------------------- | ------------------------------------------- | ------------------------ |
| Можно вызывать на [[let]]             | Нет                                         | Да                       |
| Может менять свойства [[self]]        | Да                                          | Нет                      |
| Может менять свойства других объектов | Да (если они var)                           | Да                       |
| Может быть вызван в [[init]]          | Да                                          | Да                       |
| Требуется в протоколах                | Если метод меняет self                      | Если метод только читает |
| Влияние на производительность         | Минимальное (копирование при необходимости) | Нет копирования          |

### Самые частые ошибки и ловушки с mutable в 2026

1. Забыли `mutating` в методе, который меняет свойства  
   → Ошибка компиляции: Cannot assign to property: 'self' is immutable

2. Вызывают mutating-метод на `let`-экземпляре  
   → Ошибка: Cannot use mutating member on immutable value

3. Используют `mutating` в классе (class)  
   → Ошибка: Classes cannot have mutating methods  
   → В классах свойства всегда mutable, если они var

4. mutable в протоколах без необходимости  
   → Лучше делать non-mutating, если метод не меняет self

5. mutable в SwiftUI View / @Observable  
   → Не нужно: SwiftUI и @Observable сами управляют мутабельностью

### Лучшие практики mutable в Swift 2026

- **mutating** — только когда метод **реально меняет свойства self**  
- **non-mutating** — для всех методов чтения, вычислений, геттеров  
- **[[actor]]** — не используй `mutating` (в actor всё mutable по умолчанию)  
- **Swift 6 strict concurrency** — mutable-методы в value types безопасны (копирование), но следи за Sendable  
- **[[Protocol]]** — делай методы `mutating` только если это часть семантики (например, [[Collection]] имеет `mutating func append`)  
- **Документируйте** — пиши комментарий «mutating — изменяет свойства структуры»

**Короткий девиз 2026**:
> «mutable — это когда ты говоришь: «этот метод имеет право менять меня».  
> В 2026 году это **обязательный** маркер для всех методов, которые модифицируют свойства структуры или enum.  
> Без него — компилятор не даст изменить self на let-экземпляре.»
