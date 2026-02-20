[[Xcode]] сообщает, что **вы пытаетесь завершить инициализатор (через [[return]] или конец функции), не инициализировав все stored properties класса или структуры**.

- В [[Swift]] **все свойства, не являющиеся [[Optional]] или имеющие [[default]] значение, должны быть инициализированы** перед завершением инициализатора.
    
- Если какой-либо property не получил значения → компилятор выдаёт ошибку.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Неинициализированное свойство**

```swift
class Person {
    let name: String
    let age: Int
    
    init() {
        self.name = "Alice"
        // ❌ age не инициализирован → Return from initializer without initializing all stored properties
    }
}
```

---

**Пример 2: Условная инициализация**

```swift
class Rectangle {
    let width: Double
    let height: Double
    
    init(square: Bool) {
        if square {
            width = 10
            height = 10
        }
        // ❌ если square = false, свойства не инициализированы
    }
}
```

---

**Пример 3: Инициализация с [[return]]**

```swift
struct Point {
    let x: Int
    let y: Int
    
    init() {
        return // ❌ выход до инициализации всех свойств
    }
}
```

---

### Как исправить

#### 1️⃣ Инициализировать все свойства в [[init]]

```swift
class Person {
    let name: String
    let age: Int
    
    init() {
        self.name = "Alice"
        self.age = 25 // ✅ все свойства инициализированы
    }
}
```

---

#### 2️⃣ Сделать свойства [[Optional]] или дать default значение

```swift
class Person {
    let name: String? = nil
    let age: Int = 0
    
    init() {
        // ✅ свойства имеют default значения, можно пропустить явную инициализацию
    }
}
```

---

#### 3️⃣ Убедиться, что все ветки кода инициализируют свойства

```swift
class Rectangle {
    let width: Double
    let height: Double
    
    init(square: Bool) {
        if square {
            width = 10
            height = 10
        } else {
            width = 20
            height = 10
        }
    }
}
```

---

### Резюме

- Ошибка возникает, когда **initializer завершается, не инициализировав все stored properties**.
    
- Исправляется через:
    
    1. Инициализацию всех свойств в [[init]].
        
    2. Использование default значений или optional.
        
    3. Проверку всех веток кода на инициализацию.
        
- Позволяет компилятору гарантировать корректное состояние объектов после создания.
    

---
