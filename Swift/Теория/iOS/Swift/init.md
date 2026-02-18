**`init`** — это специальная функция (инициализатор) в [[Swift]], которая отвечает за **создание и подготовку** нового экземпляра класса, структуры или перечисления.  
Она выполняется **один раз** при создании объекта и гарантирует, что все свойства будут инициализированы перед использованием.

> Проще говоря: `init` = «конструктор объекта» — место, где ты задаёшь начальное состояние.

### 1. Различия init в [[struct]] vs [[class]] vs [[enum]] (2026 актуально)

| Тип        | Автоматический memberwise init                                      | Обязательно ли явно писать init  | Можно ли иметь несколько init | Особенности                     |
| ---------- | ------------------------------------------------------------------- | -------------------------------- | ----------------------------- | ------------------------------- |
| **struct** | Да (если все свойства имеют значения по умолчанию или [[Optional]]) | Нет (если memberwise подходит)   | Да (overloading)              | Очень гибко                     |
| **class**  | Нет (кроме полностью [[default]]-значений)                          | Да (если нет default-значений)   | Да (designated + convenience) | Обязательно вызывать super.init |
| **enum**   | Да (если нет [[associated value]]s)                                 | Нет (если нет associated values) | Да                            | Часто без init вообще           |

### 2. Самые важные виды init (с примерами)

#### 2.1 Designated initializer (основной) — в классах

```swift
class Person {
    let name: String
    var age: Int
    
    // Designated init — вызывает super.init (если есть суперкласс)
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}
```

#### 2.2 Convenience initializer (вспомогательный) — в классах

```swift
class Person {
    let name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    // Convenience — вызывает другой init того же класса
    convenience init(name: String) {
        self.init(name: name, age: 0)  // ← обязательный вызов designated
    }
    
    convenience init(babyName: String) {
        self.init(name: babyName, age: 0)
    }
}

let baby = Person(babyName: "Малыш")  // age = 0
```

**Правило**:  
Все convenience init **обязательно** должны вызывать другой init (designated или convenience) в том же классе.

#### 2.3 Memberwise initializer (автоматический) — в структурах

```swift
struct Point {
    var x: Double
    var y: Double
    // Swift сам генерирует:
    // init(x: Double, y: Double) { self.x = x; self.y = y }
}

let origin = Point(x: 0, y: 0)  // автоматически сгенерированный init
```

Если добавить свой init — memberwise **исчезает** (кроме как в extension):

```swift
struct Point {
    var x: Double
    var y: Double
    
    init(x: Double, y: Double) {  // ← свой init
        self.x = x
        self.y = y
    }
}

// Point(x: 0, y: 0) — всё ещё работает
```

#### 2.4 Failable initializer (может вернуть nil)

```swift
struct User {
    let id: String
    
    init?(id: String) {
        if id.isEmpty {
            return nil
        }
        self.id = id
    }
}

let valid = User(id: "user123")     // User?
let invalid = User(id: "")          // nil
```

#### 2.5 Required init (обязательный для наследников)

```swift
class Vehicle {
    let wheels: Int
    
    required init(wheels: Int) {
        self.wheels = wheels
    }
}

class Car: Vehicle {
    let brand: String
    
    required init(wheels: Int) {  // ← обязательно
        self.brand = "Unknown"
        super.init(wheels: wheels)
    }
}
```

### 3. Лучшие практики init в Swift 2026

- **В структурах** — чаще всего полагайся на **memberwise init** (автоматический)  
- **В классах** — делай **designated init** основным, а convenience — вспомогательными  
- **Не пиши init**, если все свойства имеют значения по умолчанию или Optional — Swift сам сгенерирует  
- **Используй failable init (`init?`)** для случаев, когда создание может провалиться  
- **Required init** — только когда подклассы **обязаны** иметь этот инициализатор  
- **[[SwiftUI]] / Swift 6** — старайся делать View и ViewModel с минимальным init (по умолчанию или memberwise)  
- **Документируйте** — пиши комментарий «designated init — основной конструктор с обязательными параметрами»

**Короткий девиз 2026**:
> `init` — это «момент рождения объекта»: задай все начальные значения и подготовь состояние.  
> В 2026 году:  
> - структуры → чаще memberwise (автоматический)  
> - классы → designated + convenience  
> - используй `init?` для failable случаев  
> - required — только когда наследование требует  
> Это **основа** безопасного и предсказуемого создания объектов.
