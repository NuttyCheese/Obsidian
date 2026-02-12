## 1. Что такое sending

**Sending** — это процесс **передачи значения или объекта между различными execution context ([[Executor]]’ами)**.

- В [[Swift]] Concurrency задачи (tasks) могут выполняться на разных executor’ах: actor executor, MainActor, [[Global Executor]] или detached task.
    
- Чтобы безопасно передавать данные между этими задачами, тип должен быть **`Sendable`**.
    
- Sending гарантирует, что **данные не будут модифицироваться одновременно из разных потоков**, предотвращая гонки данных.
    

> Проще говоря: sending = «передача данных между задачами безопасно для многопоточности».

---

## 2. Связь с Sendable

- Любой объект, который передаётся между executor’ами, должен соответствовать `Sendable`.
    
- Если тип не `Sendable`, компилятор выдаст ошибку при попытке передачи его в другой task.
    

```swift
struct MyData: Sendable {
    let value: Int
}
```

- `MyData` можно безопасно **send** между задачами.
    

---

## 3. Примеры от простого к сложному

### Пример 1. Простая отправка [[Value Type]]

```swift
struct Point: Sendable {
    let x: Int
    let y: Int
}

Task.detached {
    let p = Point(x: 1, y: 2) // sending в detached task
    print(p)
}
```

- `Point` безопасно отправлен в другой executor.
    

---

### Пример 2. Sending с actor

```swift
actor Logger {
    func log(_ message: String) { print(message) }
}

Task.detached {
    let logger = Logger() // actor автоматически Sendable
    await logger.log("Привет с detached task") // безопасный sending
}
```

- Actor можно безопасно передавать между задачами благодаря internal isolation.
    

---

### Пример 3. Sending с классом через @unchecked Sendable

```swift
class Counter {
    var value = 0
}

extension Counter: @unchecked Sendable {}

Task.detached {
    let counter = Counter() // sending через @unchecked Sendable
    print(counter.value)
}
```

- Нужно быть осторожным: компилятор доверяет программисту, что класс используется безопасно.
    

---

### Пример 4. Sending массива и [[tuple]]

```swift
Task.detached {
    let numbers: [Int] = [1, 2, 3] // безопасный sending
    let point: (Int, Int) = (10, 20) // tuple тоже безопасный
    print(numbers, point)
}
```

- Все элементы должны быть Sendable, иначе компилятор выдаст ошибку.
    

---

### Пример 5. Sending с [[closure]]

```swift
Task.detached {
    let value = 42
    let closure: @Sendable () -> Int = { value } // closure безопасна для sending
    print(closure())
}
```

- Замыкания, которые используются в detached tasks или async closures, должны быть `@Sendable`.
    

---

## 4. Особенности sending

1. **Обязателен для detached tasks и async closures**
    
    - Любой объект, который передаётся в другой executor, должен быть Sendable.
        
2. **Value types проще всего send**
    
    - Структуры, [[enum]], tuple → безопасны автоматически.
        
3. **Reference types требуют осторожности**
    
    - Классы нужно делать immutable или помечать `@unchecked Sendable`.
        
4. **Actor безопасны по умолчанию**
    
    - Из-за internal executor и изоляции состояния.
        
5. **Используется для гарантии потокобезопасности**
    
    - Без sending могут возникнуть гонки данных и неопределённое поведение.
        

---

## 5. Итог

- **Sending = безопасная передача данных между задачами (tasks) и executor’ами**
    
- Требует, чтобы тип соответствовал `Sendable`
    
- [[Value Type]] → автоматически безопасны
    
- [[Reference Type]] → @unchecked Sendable или полностью потокобезопасные
    
- Actor → безопасны по умолчанию
    
- Важен для detached tasks, async closures, параллельных вычислений
    

---
