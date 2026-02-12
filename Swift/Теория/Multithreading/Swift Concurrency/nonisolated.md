## 1. Что такое `nonisolated`

**`nonisolated`** — это атрибут, который используется для **методов или свойств [[actor]]**, чтобы **они не были изолированы данным actor’ом**.

- Обычно все свойства и методы actor **автоматически изолированы**.
    
- `nonisolated` позволяет **вызывать метод напрямую, без [[await]]**, даже из внешнего контекста.
    
- Обычно используется для:
    
    - **Чистых вычислений**, которые не изменяют состояние actor
        
    - **Статических свойств/методов**, которые безопасно вызывать из любого потока
        

> Проще говоря: `nonisolated` = «этот метод безопасно вызывать извне без await, он не меняет состояние актера».

---

## 2. Основной синтаксис

```swift
actor Counter {
    var value = 0

    nonisolated func greeting() -> String {
        "Hello"
    }
}
```

- `greeting()` можно вызвать без `await`, даже снаружи actor.
    

---

## 3. Примеры от простого к сложному

### Пример 1. nonisolated метод

```swift
actor Counter {
    var value = 0

    nonisolated func greet() -> String {
        return "Привет"
    }
}

let counter = Counter()
print(counter.greet()) // ❌ без await, безопасно
```

- Здесь `greet` не изменяет состояние actor → нет необходимости в await.
    

---

### Пример 2. nonisolated static метод

```swift
actor Counter {
    static nonisolated func description() -> String {
        "Счётчик"
    }
}

print(Counter.description()) // можно вызвать из любого потока
```

---

### Пример 3. nonisolated + async function

```swift
actor Logger {
    nonisolated func info(message: String) {
        print("Лог: \(message)")
    }
}

let logger = Logger()
Task {
    logger.info(message: "Событие") // можно вызывать без await
}
```

- Метод безопасен, потому что он **не использует изолированные свойства actor**.
    

---

### Пример 4. Сравнение с обычным методом actor

```swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }

    nonisolated func greet() -> String {
        "Hello"
    }
}

let counter = Counter()

Task {
    await counter.increment() // нужно await, т.к. изолированный метод
    print(counter.greet())    // не нужен await
}
```

---

### Пример 5. Использование в делегатах и [[callback]]

```swift
protocol LoggerDelegate: AnyObject {
    func log(message: String)
}

actor Logger {
    weak var delegate: LoggerDelegate?
    
    nonisolated func notifyDelegate(message: String) {
        delegate?.log(message: message) // безопасно, не использует actor state
    }
}
```

- `nonisolated` позволяет вызывать метод напрямую в callback, не блокируя actor.
    

---

## 4. Особенности `nonisolated`

1. **Не имеет доступа к изолированным свойствам actor**
    
    - Компилятор запрещает использовать `self.value` внутри nonisolated метода
        
2. **Можно вызывать без await**
    
    - И из внешнего контекста, и из любого потока
        
3. **Используется для «чистых» методов и статических функций**
    
4. **Совместимо с протоколами**
    
    - Удобно для callback и делегатов
        

---

## 5. Итог

- `nonisolated` = метод actor **не зависит от изолированных данных**
    
- Можно вызывать без `await`
    
- Используется для статических функций, чистых вычислений, делегатов и callback
    
- Не имеет доступа к свойствам actor
    

---
