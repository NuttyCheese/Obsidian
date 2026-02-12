## 1. Что такое Isolation

**Isolation** (изоляция) — это принцип, который гарантирует, что **данные, принадлежащие конкретному контексту (например, [[actor]]), могут быть безопасно изменены только из этого контекста**.

- [[Swift]] Concurrency реализует **актерную модель (actor model)**, где каждый actor — это отдельный изолированный контекст.
    
- Все свойства и методы actor по умолчанию **изолированы**, и доступ к ним снаружи требует [[await]].
    
- Это предотвращает гонки данных и делает многопоточность безопасной.
    

> Проще говоря: isolation = «данные принадлежат только одному [[Executor]]’у и нельзя менять их снаружи напрямую».

---

## 2. Виды изоляции

1. **Actor isolation**
    
    - Каждый `actor` имеет **собственный executor**.
        
    - Доступ к его данным снаружи → через `await`.
        
2. **MainActor isolation**
    
    - Все свойства и методы помеченные [[@MainActor]] → выполняются на **главном потоке**.
        
    - Гарантирует безопасное обновление UI.
        
3. **[[Global Executor]] / Detached task**
    
    - Нет специфической изоляции → работа с общими данными требует синхронизации.
        
4. **Parameter isolation (`isolated`)**
    
    - Аргументы функции могут быть помечены как `isolated`, если они **уже находятся в безопасном контексте**.
        

---

## 3. Основные правила

1. **Доступ к изолированным данным actor снаружи требует await**
    

```swift
actor Counter {
    var value = 0
}

let counter = Counter()
Task {
    await counter.value // доступ к изолированным данным через await
}
```

2. **Внутри actor await не нужен**
    

```swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1 // безопасно, т.к. мы уже в executor
    }
}
```

3. **Параметры `isolated` позволяют работать с данными безопасно вне await**
    

```swift
actor Counter {
    var value = 0
    
    func add(delta: isolated Int) {
        value += delta
    }
}
```

4. **MainActor обеспечивает изоляцию на главном потоке**
    

```swift
@MainActor
func updateUI() {
    // безопасное изменение UI
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Actor isolation

```swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}

let counter = Counter()

Task {
    await counter.increment() // переключение на executor Counter
}
```

---

### Пример 2. MainActor isolation

```swift
@MainActor
func updateLabel() {
    print("Изменение UI на главном потоке")
}

Task {
    await updateLabel()
}
```

---

### Пример 3. Parameter isolation

```swift
actor Counter {
    var value = 0
    
    func add(delta: isolated Int) {
        value += delta
    }
}

Task {
    let counter = Counter()
    await counter.add(delta: 5)
}
```

---

### Пример 4. Closure с isolated

```swift
actor Counter {
    var value = 0
    
    func incrementWithClosure(_ closure: @Sendable (isolated Int) -> Int) {
        value += closure(value)
    }
}

Task {
    let counter = Counter()
    await counter.incrementWithClosure { isolatedValue in
        return isolatedValue + 10
    }
}
```

---

### Пример 5. [[AsyncSequence]] с actor isolation

```swift
actor Logger {
    var messages: [String] = []

    func addMessage(_ message: String) {
        messages.append(message)
    }

    func stream() -> AsyncStream<String> {
        AsyncStream { continuation in
            Task {
                for msg in await messages {
                    continuation.yield(msg)
                }
                continuation.finish()
            }
        }
    }
}

let logger = Logger()
Task {
    await logger.addMessage("Сообщение 1")
    for await msg in await logger.stream() {
        print(msg)
    }
}
```

---

## 5. Итог

- **Isolation = изоляция данных для безопасности в многопоточности**
    
- Actor = изолированный executor, MainActor = главный поток, isolated параметры = безопасный доступ
    
- Для доступа к изолированным данным извне → `await`
    
- Внутри изолированного контекста → await не нужен
    
- Позволяет избежать гонок данных и блокировок
    

---
