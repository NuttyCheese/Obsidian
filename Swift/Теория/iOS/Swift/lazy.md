**`lazy`** — это модификатор свойства, который **откладывает его инициализацию до первого обращения**.

- Свойство не создаётся при создании объекта
    
- Инициализация происходит **только при первом доступе**
    
- Работает **только с переменными ([[var]])**, нельзя использовать с [[let]]
    

> Проще говоря: `lazy` = «создаём свойство только тогда, когда оно реально нужно».

---

## 2. Основные термины

| Термин                                  | Описание                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------- |
| **Lazy property**                       | Свойство, которое создаётся только при первом использовании                |
| **Initialization**                      | Присвоение значения свойству                                               |
| **[[Value Type]] / [[Reference Type]]** | Lazy свойство может быть любым типом (структура, класс, коллекция)         |
| **Self**                                | В lazy свойствах можно использовать [[self]], потому что объект уже создан |
| **Memory optimization**                 | Lazy помогает экономить память, если свойство редко используется           |

---

## 3. Основной синтаксис

```swift
class DataManager {
    lazy var data = loadData()

    func loadData() -> [String] {
        print("Loading data...")
        return ["One", "Two", "Three"]
    }
}

let manager = DataManager()
// пока ничего не загружено
print(manager.data) 
// Output:
// Loading data...
// ["One", "Two", "Three"]
```

- `lazy var data = loadData()` → значение вычисляется только при первом вызове `manager.data`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое lazy свойство

```swift
class Person {
    var name: String
    lazy var greeting: String = "Hello, \(name)!"

    init(name: String) {
        self.name = name
    }
}

let alice = Person(name: "Alice")
print(alice.greeting) // Hello, Alice!
```

- Свойство `greeting` инициализируется **только при первом вызове**
    

---

### Пример 2. Lazy с вычислением

```swift
struct Circle {
    var radius: Double
    lazy var area: Double = {
        return Double.pi * radius * radius
    }()
}

var circle = Circle(radius: 5)
print(circle.area) // 78.53981633974483
```

- Можно использовать **замыкание для вычисления значения**
    

---

### Пример 3. Lazy с коллекцией

```swift
class Numbers {
    lazy var squares: [Int] = (1...5).map { $0 * $0 }
}

let nums = Numbers()
// пока массив не создан
print(nums.squares) // [1, 4, 9, 16, 25]
```

- Полезно для больших коллекций или сложных вычислений
    

---

### Пример 4. Lazy с self

```swift
class A {
    var value = 10
    lazy var doubleValue = value * 2
}

let a = A()
a.value = 15
print(a.doubleValue) // 30
```

- Lazy свойство использует **текущее состояние self** в момент инициализации
    

---

### Пример 5. Lazy и многопотоковость

```swift
class Config {
    lazy var settings: [String: Any] = {
        print("Loading settings...")
        return ["theme": "dark", "version": 1.0]
    }()
}

let config = Config()
DispatchQueue.global().async {
    print(config.settings)
}
```

- Lazy свойства **создаются один раз**, даже при многопоточном доступе (Swift гарантирует безопасную инициализацию)
    

---

## 5. Особенности lazy

1. Работает **только с var**, нельзя с let
    
2. Инициализация происходит **один раз при первом обращении**
    
3. Можно использовать **замыкания для вычисления**
    
4. Подходит для **дорогих или редко используемых свойств**
    
5. Можно безопасно использовать **self внутри lazy**
    

---

## 6. Итог

- **`lazy`** = отложенная инициализация свойства, создаётся только при первом обращении
    
- Экономит **память и ресурсы**
    
- Часто используется для **коллекций, вычислений, больших объектов, зависимых от self**
    

---
