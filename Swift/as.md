`as` — оператор приведения типов в [[Swift]]. Используется для явного приведения к более широкому типу, к протоколу или для паттерн-матчинга. Есть варианты: `as`, `as?`, `as!`.

### Стек

**Swift Language (часть стандартного языка, не фреймворк).**

---

```swift
// Пример 1 — простое приведение к типу родителя
let number: Int = 10
let anyValue: Any = number as Any   // приведение к Any
```

```swift
// Пример 2 — приведение к протоколу
protocol Animal { }
struct Dog: Animal { }

let dog = Dog()
let animal: Animal = dog as Animal
```

```swift
// Пример 3 — условное приведение as?
let value: Any = "Hello"

if let text = value as? String {
    print("Строка:", text)
}
```

```swift
// Пример 4 — принудительное приведение as!
let value: Any = 42
let intValue = value as! Int   // если тип неверный — краш
```

```swift
// Пример 5 — приведение типов в иерархии классов
class Vehicle { }
class Car: Vehicle { }

let car = Car()
let vehicle: Vehicle = car

if let castedCar = vehicle as? Car {
    print("Это машина:", castedCar)
}
```

```swift
// Пример 6 — использование as в switch (pattern matching)
let items: [Any] = [1, "text", 3.14]

for item in items {
    switch item {
    case let number as Int:
        print("Int:", number)
    case let text as String:
        print("String:", text)
    default:
        print("Other type")
    }
}
```