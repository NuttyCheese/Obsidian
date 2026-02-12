## 1. Что такое `init`

**`init`** — это **инициализатор**, специальная функция, которая:

- Создаёт новый экземпляр **класса, структуры или перечисления**
    
- Устанавливает **начальные значения свойств**
    
- Может быть **несколько инициализаторов с разными параметрами** (overloading)
    
- В классах используется вместе с [[deinit]] для освобождения ресурсов
    

> Проще говоря: `init` = «функция, которая создаёт объект и задаёт начальные значения».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Initializer**|Функция `init`, создающая объект или структуру|
|**Default initializer**|Автоматический инициализатор для всех свойств без параметров|
|**Memberwise initializer**|Автоматический инициализатор для структур, принимает все свойства|
|**Designated initializer**|Основной инициализатор класса, вызывающий super.init при наследовании|
|**Convenience initializer**|Вспомогательный инициализатор, вызывает другой инициализатор в том же классе|

---

## 3. Основной синтаксис

```swift
struct Point {
    var x: Int
    var y: Int

    init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}

let p = Point(x: 10, y: 20)
```

- `self.x = x` → присвоение значения свойству объекта
    
- `self` указывает на **текущий экземпляр**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Структура с init

```swift
struct Person {
    var name: String
    var age: Int
}

let alice = Person(name: "Alice", age: 25)
print(alice.name, alice.age) // Alice 25
```

- Используется **memberwise initializer**, который [[Swift]] генерирует автоматически
    

---

### Пример 2. Класс с init

```swift
class Car {
    var model: String
    var year: Int

    init(model: String, year: Int) {
        self.model = model
        self.year = year
    }
}

let car = Car(model: "Tesla", year: 2023)
print(car.model, car.year) // Tesla 2023
```

- Для классов необходимо явно создавать инициализатор, если нет значений по умолчанию
    

---

### Пример 3. Convenience init (класс)

```swift
class Rectangle {
    var width: Double
    var height: Double

    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }

    convenience init(size: Double) {
        self.init(width: size, height: size)
    }
}

let square = Rectangle(size: 10)
print(square.width, square.height) // 10 10
```

- `convenience init` → вспомогательный, вызывает основной `init`
    

---

### Пример 4. [[Optional]] свойства и init

```swift
struct Book {
    var title: String
    var author: String?
}

let book1 = Book(title: "Swift Guide", author: nil)
let book2 = Book(title: "Swift Guide", author: "John Doe")
```

- Optional свойства **не требуют значения при инициализации**
    

---

### Пример 5. Init с логикой

```swift
struct Circle {
    var radius: Double
    var area: Double

    init(radius: Double) {
        self.radius = radius
        self.area = Double.pi * radius * radius
    }
}

let circle = Circle(radius: 5)
print(circle.area) // 78.53981633974483
```

- Init может выполнять **вычисления и дополнительную логику** при создании объекта
    

---

## 5. Особенности init

1. Используется для **инициализации всех свойств объекта или структуры**
    
2. Можно иметь **несколько инициализаторов** (overloading)
    
3. В классах есть **designated** и **convenience** init
    
4. Свойства Optional **необязательны для инициализации**
    
5. Init может содержать **любую логику** до создания объекта
    

---

## 6. Итог

- **`init`** = функция, создающая объект и задающая его свойства
    
- Обеспечивает **безопасное создание экземпляров**
    
- Может быть **несколько инициализаторов** с разными параметрами и логикой
    
- В классах используется совместно с **convenience init** и наследованием
    

---
