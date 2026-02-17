**`class`** — это **ссылочный тип ([[Reference Type]])**, используемый для инкапсуляции состояния и поведения объектов.

- Хранит **состояние через свойства**
    
- Инкапсулирует **поведение через методы**
    
- Позволяет использовать **наследование** и **полиморфизм**
    

> Проще говоря: class = «чёртёж объекта, который можно создать несколько раз и передавать по ссылке».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Reference type**|Объекты класса передаются по ссылке, а не копируются|
|**Instance**|Конкретный объект класса|
|**Properties**|Свойства (данные) класса|
|**Methods**|Методы (функции) класса|
|**Inheritance**|Наследование от другого класса|
|**Deinitializer**|`deinit` — освобождение ресурсов при уничтожении объекта|
|**Final**|Запрещает наследование класса|
|**Type casting**|Приведение объекта к другому типу в иерархии|

---

## 3. Основной синтаксис

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func greet() {
        print("Hello, my name is \(name)")
    }
}

let person = Person(name: "Alice", age: 30)
person.greet() // Hello, my name is Alice
```

- [[init]] — конструктор класса
    
- Созданный объект — **instance**
    
- Объекты передаются по **ссылке**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший класс

```swift
class Counter {
    var count = 0
    func increment() {
        count += 1
    }
}

let counter = Counter()
counter.increment()
print(counter.count) // 1
```

- Создан объект, свойства и методы работают через ссылку
    

---

### Пример 2. Наследование

```swift
class Animal {
    func speak() {
        print("Animal sound")
    }
}

class Dog: Animal {
    override func speak() {
        print("Woof!")
    }
}

let dog = Dog()
dog.speak() // Woof!
```

- Наследование и **[[override]]** позволяют изменять поведение методов
    

---

### Пример 3. Сравнение ссылок

```swift
let a = Counter()
let b = a
b.increment()
print(a.count) // 1, ссылка одна и та же
```

- `class` — reference type, изменения через одну ссылку видны через другие
    

---

### Пример 4. Deinitializer

```swift
class Temp {
    deinit {
        print("Temp deallocated")
    }
}

var temp: Temp? = Temp()
temp = nil // Temp deallocated
```

- [[deinit]] вызывается при освобождении памяти объекта
    

---

### Пример 5. Полиморфизм и типы

```swift
class Shape {
    func area() -> Double { 0 }
}

class Circle: Shape {
    var radius: Double
    init(radius: Double) { self.radius = radius }
    override func area() -> Double { .pi * radius * radius }
}

class Square: Shape {
    var side: Double
    init(side: Double) { self.side = side }
    override func area() -> Double { side * side }
}

let shapes: [Shape] = [Circle(radius: 2), Square(side: 3)]
for shape in shapes {
    print(shape.area()) // 12.566..., 9.0
}
```

- Полиморфизм позволяет работать с массивом разных объектов через базовый тип
    

---

## 5. Особенности `class`

1. **Reference type** — объекты передаются по ссылке
    
2. **Наследование** — можно создавать иерархию классов
    
3. **Deinitializer** — `deinit` для освобождения ресурсов
    
4. **final** — запрещает наследование
    
5. **Типы и приведение** — `is` / `as?` / `as!` для проверки и приведения типов
    
6. **Сравнение объектов** — через ссылку (`===`), а не по значению (`==`)
    

---

## 6. Итог

- `class` = объектно-ориентированный тип с ссылочной семантикой
    
- Поддерживает **состояние, методы, наследование, полиморфизм**
    
- Reference type = одна и та же память для всех ссылок на объект
    
- Отличается от [[struct]] ([[Value Type]]) тем, что struct копируется при присваивании
    

---
