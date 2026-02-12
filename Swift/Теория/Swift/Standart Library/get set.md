## 1. Что такое `get set`

**`get set`** — это часть **вычисляемого свойства**, которая определяет:

- **[[Swift/Теория/Swift/Standart Library/get]]** — как получить значение
    
- **[[Set]]** — как установить новое значение
    
- ==Используется только с [[var]]==
    
- Позволяет вычислять значение **на лету** и управлять изменением зависимых свойств
    

> Проще говоря: `get set` = «как прочитать и записать значение свойства».

---

## 2. Основные термины

| Термин                  | Описание                                                                       |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Computed Property**   | Свойство, которое вычисляется на лету через `get` и/или `set`                  |
| **Read-write Property** | Свойство, которое можно читать и изменять                                      |
| **newValue**            | Специальная переменная в `set`, содержит новое присваиваемое значение          |
| **Property Observer**   | [[willSet]] и [[didSet]] работают только с хранимыми свойствами, не с computed |

---

## 3. Основной синтаксис

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
print(circle.diameter) // 10
circle.diameter = 20
print(circle.radius)   // 10
```

- `get` вычисляет значение на основе `radius`
    
- `set` обновляет `radius` через `newValue`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое read-write свойство

```swift
struct Rectangle {
    var width: Double
    var height: Double
    var area: Double {
        get {
            return width * height
        }
        set {
            width = sqrt(newValue) // меняем width для простоты
            height = sqrt(newValue)
        }
    }
}

var rect = Rectangle(width: 3, height: 4)
print(rect.area) // 12
rect.area = 16
print(rect.width, rect.height) // 4.0 4.0
```

- Свойство можно **читать и писать**
    

---

### Пример 2. Сокращённая запись get/set

```swift
struct Square {
    var side: Double
    var perimeter: Double {
        get { side * 4 }
        set { side = newValue / 4 }
    }
}

var square = Square(side: 5)
print(square.perimeter) // 20
square.perimeter = 40
print(square.side)      // 10
```

- Однострочные вычисления через `{ get { … } set { … } }`
    

---

### Пример 3. get/set с [[Optional]]

```swift
struct User {
    var firstName: String?
    var lastName: String?
    
    var fullName: String {
        get { "\(firstName ?? "No") \(lastName ?? "Name")" }
        set {
            let components = newValue.split(separator: " ")
            firstName = String(components.first ?? "No")
            lastName = String(components.last ?? "Name")
        }
    }
}

var user = User(firstName: "Alice", lastName: nil)
print(user.fullName) // Alice Name
user.fullName = "Bob Marley"
print(user.firstName!, user.lastName!) // Bob Marley
```

- Сеттер разбивает строку и обновляет свойства
    

---

### Пример 4. get/set с логикой проверки

```swift
struct Temperature {
    var celsius: Double
    var fahrenheit: Double {
        get { celsius * 9 / 5 + 32 }
        set {
            if newValue >= -459.67 { // проверка на абсолютный ноль
                celsius = (newValue - 32) * 5 / 9
            }
        }
    }
}

var temp = Temperature(celsius: 25)
print(temp.fahrenheit) // 77
temp.fahrenheit = -500 // не изменится
print(temp.celsius)    // 25
```

- Можно добавить **валидацию в сеттер**
    

---

### Пример 5. Класс с get/set

```swift
class Circle {
    var radius: Double
    var diameter: Double {
        get { radius * 2 }
        set { radius = newValue / 2 }
    }
    
    init(radius: Double) { self.radius = radius }
}

let circle = Circle(radius: 5)
print(circle.diameter) // 10
circle.diameter = 20
print(circle.radius)   // 10
```

- `get/set` одинаково работает в **классах и структурах**
    

---

## 5. Особенности get/set

1. **get** всегда обязателен, **set** — по желанию
    
2. `set` использует `newValue` для присваивания
    
3. Свойства могут быть read-only (только `get`) или read-write (`get set`)
    
4. Можно добавлять **валидацию или вычисления** в сеттер
    
5. Работает с **структурами, классами и optional**
    

---

## 6. Итог

- **get set** = вычисляемое свойство с чтением и записью
    
- Позволяет **инкапсулировать логику получения и установки значения**
    
- Можно использовать:
    
    - Для вычисляемых значений (`diameter`, `area`)
        
    - Для строк и optional
        
    - Для классов и структур
        
    - Для проверки/валидации внутри set
        

---
