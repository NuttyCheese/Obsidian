#crash #warning #xcode #swift
### Что это значит

[[Xcode]] предупреждает, что **тип не соответствует протоколу [[Sendable]]**, необходимому для безопасной работы с **параллельными или асинхронными задачами** ([[async]]/[[await]], [[Task]], [[@MainActor]] и т.д.).

- В [[Swift]] Concurrency типы, передаваемые между потоками, **должны быть `Sendable`**, чтобы гарантировать потокобезопасность.
    
- `Sendable` обеспечивает, что значения **не будут изменяться одновременно из разных задач**.
    
- Типы, которые **содержат изменяемые свойства** или **непотокобезопасные объекты**, по умолчанию **не conform** `Sendable`.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Структура с [[var]] свойствами**

```swift
struct User {
    var name: String
    var age: Int
}

func processUser() async {
    let user = User(name: "Alice", age: 30)
    Task {
        print(user) // ⚠️ Type 'User' does not conform to protocol 'Sendable'
    }
}
```

- `User` содержит изменяемые свойства (`var`) → не является безопасным для передачи между потоками.
    

---

**Пример 2: Класс с ссылочными типами**

```swift
class Counter {
    var value = 0
}

func incrementCounter() async {
    let counter = Counter()
    Task {
        counter.value += 1 // ⚠️ Type 'Counter' does not conform to protocol 'Sendable'
    }
}
```

- Ссылочные типы (классы) **не thread-safe по умолчанию** → нельзя безопасно передавать в Task.
    

---

### Как исправить

#### 1️⃣ Сделать структуру `Sendable` и неизменяемой

```swift
struct User: Sendable {
    let name: String
    let age: Int
}
```

- Используем [[let]] вместо `var` → гарантируем immutability.
    

---

#### 2️⃣ Использовать `@unchecked Sendable` (только если уверены в безопасности)

```swift
class Counter: @unchecked Sendable {
    var value = 0
}
```

- Говорим компилятору «я гарантирую потокобезопасность», но **это ложится на ответственность разработчика**.
    

---

#### 3️⃣ Использовать [[actor]] для классов

```swift
actor Counter {
    var value = 0
    func increment() { value += 1 }
}

func incrementCounter() async {
    let counter = Counter()
    await counter.increment() // ✅ безопасно
}
```

- `actor` гарантирует потокобезопасность для классовых свойств → автоматически conform `Sendable`.
    

---

### Резюме

- Предупреждение появляется при **передаче типов между concurrent задачами**, которые не являются потокобезопасными (`Sendable`).
    
- Исправляется:
    
    1. Сделать тип **immutable struct** и conform `Sendable`.
        
    2. Использовать **actor** для классов.
        
    3. В крайнем случае — `@unchecked Sendable`.
        
- Обеспечивает безопасное использование Swift Concurrency и предотвращает [[race condition]].
    

---
