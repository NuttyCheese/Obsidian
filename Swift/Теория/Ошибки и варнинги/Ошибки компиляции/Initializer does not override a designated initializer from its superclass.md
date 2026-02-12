#crash #fata_error #xcode #swift
### Что это значит

[[Xcode]] сообщает, что **инициализатор в подклассе помечен как [[override]]**, но **не соответствует ни одному designated initializer суперкласса**.

- В [[Swift]] есть два типа инициализаторов:
    
    1. **Designated initializer** — основной инициализатор класса.
        
    2. **Convenience initializer** — вспомогательный, вызывает designated initializer.
        
- **Правила override:**
    
    - Можно override только **designated initializer** суперкласса.
        
    - Если вы пометили `override`, а метод не является designated initializer суперкласса → ошибка.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Override init с несоответствующей сигнатурой**

```swift
class Animal {
    init(name: String) { }
}

class Dog: Animal {
    override init() { } // ❌ Initializer does not override a designated initializer from its superclass
}
```

- `Animal` имеет designated initializer `init(name:)`.
    
- `Dog` пытается override `init()` → такой инициализатор в суперклассе нет.
    

---

**Пример 2: Convenience [[init]], но помечен override**

```swift
class Vehicle {
    init(model: String) { }
}

class Car: Vehicle {
    override convenience init(model: String) { } // ❌ ошибка
}
```

- Convenience init нельзя помечать `override`, override только для designated init.
    

---

### Как исправить

#### 1️⃣ Подстроить сигнатуру под designated initializer

```swift
class Dog: Animal {
    override init(name: String) {
        super.init(name: name) // ✅ корректно override
    }
}
```

---

#### 2️⃣ Если нужен другой инициализатор — сделать convenience

```swift
class Dog: Animal {
    convenience init() {
        self.init(name: "Unknown") // ✅ вызывает designated initializer
    }
}
```

---

#### 3️⃣ Проверить суперкласс

- Убедитесь, что вы override **существующий designated initializer**, а не создаёте новый.
    
- Для классов с [[NSObject]] иногда требуется явно вызвать `super.init()`.
    

---

### Резюме

- Ошибка возникает, когда **подкласс пытается override инициализатор, которого нет в суперклассе**.
    
- Исправляется через:
    
    1. Использование правильной сигнатуры override.
        
    2. Создание convenience initializer вместо override.
        
    3. Проверку designated инициализаторов суперкласса.
        
- Позволяет поддерживать правильную цепочку инициализации и избегать runtime crash.
    

---
