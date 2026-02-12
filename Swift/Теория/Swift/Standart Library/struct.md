## 1. Что такое `struct`

**`struct`** — это **тип значения ([[Value Type]])** в [[Swift]], который позволяет объединять **свойства (properties) и методы (methods)** в один логический блок.

- **Value type** → копируется при присваивании или передаче в функцию
    
- Может иметь **инициализаторы, методы, computed properties, subscripts**
    
- Часто используется для **легких объектов**, моделей данных, конфигураций
    

> Проще говоря: struct = «контейнер с данными и функциями, который копируется при передаче».

---

## 2. Основные термины

| Термин                   | Описание                                                                   |
| ------------------------ | -------------------------------------------------------------------------- |
| **Value type**           | Копия создаётся при присваивании или передаче                              |
| **Property**             | Переменная или константа внутри struct                                     |
| **Method**               | Функция внутри struct                                                      |
| **Computed property**    | Свойство, которое вычисляется при обращении                                |
| **Mutating method**      | Метод, который может изменять свойства struct                              |
| **Initializer**          | Функция для создания экземпляра struct                                     |
| **Protocol conformance** | Struct может соответствовать протоколам ([[Equatable]], [[Codable]] и др.) |

---

## 3. Основной синтаксис

```swift
struct Point {
    var x: Int
    var y: Int
}

let p1 = Point(x: 10, y: 20)
```

- `Point` — struct с двумя свойствами
    
- Присваивание `let p1 = ...` делает экземпляр **immutable**, для изменения нужно [[var]]
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший struct

```swift
struct Point {
    var x: Int
    var y: Int
}

var p = Point(x: 1, y: 2)
p.x = 5
print(p) // Point(x: 5, y: 2)
```

- Mutable экземпляр можно изменять
    

---

### Пример 2. Computed property

```swift
struct Rectangle {
    var width: Double
    var height: Double
    
    var area: Double {
        return width * height
    }
}

let rect = Rectangle(width: 3, height: 4)
print(rect.area) // 12
```

- `area` вычисляется при каждом обращении
    

---

### Пример 3. Mutating method

```swift
struct Counter {
    var count = 0
    
    mutating func increment() {
        count += 1
    }
}

var counter = Counter()
counter.increment()
print(counter.count) // 1
```

- Методы, которые изменяют свойства struct, должны быть **mutating**
    

---

### Пример 4. Initializer

```swift
struct Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

let alice = Person(name: "Alice", age: 30)
print(alice.name, alice.age)
```

- Можно создавать **кастомные инициализаторы**
    
- Swift создаёт **memberwise initializer** автоматически, если не объявлен свой
    

---

### Пример 5. Protocol conformance

```swift
struct Point: Equatable, Codable {
    var x: Int
    var y: Int
}

let p1 = Point(x: 1, y: 2)
let p2 = Point(x: 1, y: 2)
print(p1 == p2) // true
```

- Struct может соответствовать протоколам
    
- `Equatable` позволяет сравнивать экземпляры
    
- `Codable` позволяет сериализовать и десериализовать
    

---

### Пример 6. Nested struct и методы

```swift
struct Circle {
    struct Point {
        var x: Double
        var y: Double
    }
    
    var center: Point
    var radius: Double
    
    func circumference() -> Double {
        return 2 * .pi * radius
    }
}

let circle = Circle(center: Circle.Point(x: 0, y: 0), radius: 5)
print(circle.circumference()) // 31.4159...
```

- Struct можно **вложить** в другой struct
    
- Методы могут работать с данными экземпляра
    

---

## 5. Особенности struct

1. **Value type** → копируется при присваивании
    
2. Mutable экземпляры требуют `var`, immutable — [[let]]
    
3. Поддерживает **свойства, методы, computed properties, mutating methods**
    
4. Может соответствовать протоколам (`Equatable`, `Codable`, `Hashable`)
    
5. Отличается от классов тем, что **не поддерживает наследование**, но может использовать **composition**
    

---

## 6. Итог

- **Struct** = тип значения с данными и функциями
    
- Копируется при присваивании → безопасно для многопоточности
    
- Поддерживает **computed properties, mutating methods, initializers, protocol conformance**
    
- Используется для **легких объектов, моделей, конфигураций**
    

---
