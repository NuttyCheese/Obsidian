#crash #fata_error #xcode #swift
### Что это значит

[[Xcode]] сообщает, что **вы переопределяете метод, свойство или инициализатор суперкласса**, но **не пометили его ключевым словом [[override]]**.

- В [[Swift]] **все переопределения методов или свойств суперкласса должны явно использовать `override`**.
    
- Это необходимо для:
    
    1. Явного указания намерения разработчика.
        
    2. Предотвращения случайного скрытия методов суперкласса.
        
- Ошибка возникает, когда вы просто пишете метод с такой же сигнатурой, как у суперкласса, но без `override`.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Метод суперкласса**

```swift
class Animal {
    func speak() {
        print("Animal sound")
    }
}

class Dog: Animal {
    func speak() { // ❌ Overriding declaration requires an 'override' keyword
        print("Bark")
    }
}
```

- Метод `speak()` уже существует в `Animal`, нужно использовать `override`.
    

---

**Пример 2: Свойство суперкласса**

```swift
class Vehicle {
    var speed: Int { 0 }
}

class Car: Vehicle {
    var speed: Int { 200 } // ❌ Overriding declaration requires an 'override' keyword
}
```

- Свойство `speed` существует в суперклассе → необходимо `override`.
    

---

**Пример 3: Инициализатор суперкласса**

```swift
class Person {
    init(name: String) { }
}

class Student: Person {
    init(name: String) { } // ❌ Overriding declaration requires an 'override' keyword
}
```

- Переопределение designated initializer требует `override` (если суперкласс не помечен `required`).
    

---

### Как исправить

#### 1️⃣ Добавить ключевое слово `override`

```swift
class Dog: Animal {
    override func speak() {
        print("Bark")
    }
}
```

```swift
class Car: Vehicle {
    override var speed: Int { 200 }
}
```

---

#### 2️⃣ Для инициализаторов использовать `override` или `required`

```swift
class Student: Person {
    override init(name: String) {
        super.init(name: name)
    }
}
```

---

### Резюме

- Ошибка возникает, когда **метод, свойство или инициализатор суперкласса переопределяется без `override`**.
    
- Исправляется через:
    
    1. Добавление `override` перед методом, свойством или инициализатором.
        
    2. Проверку цепочки наследования и сигнатур методов.
        
- Позволяет компилятору различать новые методы и методы, переопределяющие суперкласс, предотвращая случайное скрытие функций.
    

---
