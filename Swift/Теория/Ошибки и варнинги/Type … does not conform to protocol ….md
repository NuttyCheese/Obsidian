[[Xcode]] сообщает, что **тип ([[struct]], [[class]] или [[enum]]) заявлен как соответствующий протоколу**, но **не реализует все требования этого протокола**.

- Протоколы могут требовать:
    
    1. Методы и свойства.
        
    2. Инициализаторы.
        
    3. Ассоциированные типы ([[AssociatedType]]).
        
- Если тип **не реализует один или несколько обязательных элементов протокола**, компилятор выдаёт ошибку.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Пропущенный метод**

```swift
protocol Greetable {
    func greet()
}

struct Person: Greetable {
    // ❌ Type 'Person' does not conform to protocol 'Greetable'
}
```

- Метод `greet()` не реализован.
    

---

**Пример 2: Пропущенное свойство**

```swift
protocol Identifiable {
    var id: String { get }
}

struct User: Identifiable {
    var name: String
    // ❌ Type 'User' does not conform to protocol 'Identifiable'
}
```

- Свойство `id` не реализовано.
    

---

**Пример 3: Несовпадение сигнатуры метода**

```swift
protocol Summable {
    func sum(a: Int, b: Int) -> Int
}

struct Calculator: Summable {
    func sum(x: Int, y: Int) -> Int { // ❌ Type 'Calculator' does not conform to protocol 'Summable'
        return x + y
    }
}
```

- Сигнатура метода должна точно совпадать (`a` и `b`).
    

---

**Пример 4: Пропущенный required инициализатор для класса**

```swift
protocol Nameable {
    init(name: String)
}

class Person: Nameable {
    var name: String
    init() { self.name = "Unknown" } // ❌ Type 'Person' does not conform to protocol 'Nameable'
}
```

- Не реализован required init с сигнатурой `init(name: String)`.
    

---

### Как исправить

#### 1️⃣ Реализовать все обязательные методы и свойства

```swift
struct Person: Greetable {
    func greet() {
        print("Hello")
    }
}
```

---

#### 2️⃣ Проверить сигнатуры методов и свойств

```swift
struct Calculator: Summable {
    func sum(a: Int, b: Int) -> Int {
        return a + b
    }
}
```

---

#### 3️⃣ Для [[required init]] в классе

```swift
class Person: Nameable {
    var name: String
    required init(name: String) {
        self.name = name
    }
}
```

---

### Резюме

- Ошибка возникает, когда **тип не реализует все требования протокола**.
    
- Исправляется через:
    
    1. Реализацию всех методов и свойств протокола.
        
    2. Проверку сигнатур и имен аргументов.
        
    3. Использование `required` для инициализаторов в классах.
        
- Позволяет компилятору гарантировать полное соответствие протоколу и корректную работу с [[generic]] или polymorphism.
    

---
