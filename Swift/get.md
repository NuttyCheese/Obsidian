**`get`** — это часть **вычисляемого свойства (computed property)**, которая **возвращает значение**.

- Используется вместе с [[var]]
    
- Может иметь только `get` (только чтение) или вместе с [[Set]] (чтение/запись)
    
- Позволяет **логически вычислять значение** без хранения его напрямую
    

> Проще говоря: `get` = «как получить значение свойства».

---

## 2. Основные термины

| Термин                 | Описание                                                                    |
| ---------------------- | --------------------------------------------------------------------------- |
| **Computed Property**  | Свойство, которое вычисляется на лету через `get` (и опционально `set`)     |
| **Read-only Property** | Свойство с `get`, которое нельзя изменять                                   |
| **Setter (`set`)**     | Дополняет `get`, чтобы сделать свойство изменяемым                          |
| **Property Observer**  | Наблюдатели [[willSet]] и [[didSet]] работают только с хранимыми свойствами |

---

## 3. Основной синтаксис

```swift
struct Circle {
    var radius: Double
    var diameter: Double {
        get {
            return radius * 2
        }
    }
}

let circle = Circle(radius: 5)
print(circle.diameter) // 10
```

- `diameter` вычисляется на лету через `get`
    
- Значение **не хранится**, только вычисляется
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое read-only свойство

```swift
struct Rectangle {
    var width: Double
    var height: Double
    var area: Double {
        get {
            return width * height
        }
    }
}

let rect = Rectangle(width: 3, height: 4)
print(rect.area) // 12
```

- `area` вычисляется, **не хранится**
    

---

### Пример 2. Сокращённая запись get

```swift
struct Square {
    var side: Double
    var perimeter: Double {
        side * 4
    }
}

let square = Square(side: 5)
print(square.perimeter) // 20
```

- Можно опустить `get {}` для **однострочного вычисления**
    

---

### Пример 3. get + set

```swift
struct Circle {
    var radius: Double
    var diameter: Double {
        get {
            return radius * 2
        }
        set {
            radius = newValue / 2
        }
    }
}

var circle = Circle(radius: 5)
circle.diameter = 20
print(circle.radius) // 10
```

- `get` возвращает, `set` изменяет зависимое свойство
    
- `newValue` — ключевое слово для нового значения
    

---

### Пример 4. get с логикой

```swift
struct Temperature {
    var celsius: Double
    var fahrenheit: Double {
        get {
            return celsius * 9 / 5 + 32
        }
    }
}

let temp = Temperature(celsius: 25)
print(temp.fahrenheit) // 77
```

- Свойство вычисляется через формулу
    
- Только для чтения
    

---

### Пример 5. get с [[Optional]]

```swift
struct User {
    var firstName: String?
    var lastName: String?
    
    var fullName: String {
        get {
            return "\(firstName ?? "No") \(lastName ?? "Name")"
        }
    }
}

let user = User(firstName: "Alice", lastName: nil)
print(user.fullName) // Alice Name
```

- Можно безопасно обрабатывать **Optional значения**
    

---

## 5. Особенности get

1. Используется **только с вычисляемыми свойствами**
    
2. Может быть **read-only** или вместе с `set`
    
3. Позволяет вычислять значения **на лету, без хранения**
    
4. Можно писать в **сокращённой форме для одной строки**
    
5. Позволяет **инкапсулировать логику вычисления**
    

---

## 6. Итог

- **get** = способ получить значение вычисляемого свойства
    
- Используется для **read-only или computed properties**
    
- Можно комбинировать с `set` для изменяемых свойств
    
- Позволяет **сохранять чистоту кода и инкапсуляцию логики**
    

---
