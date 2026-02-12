## 1. Что такое `Task`

**`Task`** — это объект, который **создаёт и управляет асинхронной задачей**.

- Позволяет запускать код в **асинхронном контексте** без блокировки потока.
    
- Задачи могут выполняться:
    
    - на текущем [[Executor]] (например, [[actor]] или MainActor)
        
    - на глобальном executor (фоновые потоки) через `Task.detached`
        

> Проще говоря: Task = «контейнер для асинхронной работы».

---

## 2. Основные виды задач

1. **Нормальный Task**
    

```swift
Task {
    // выполняется в текущем executor
}
```

2. **Detached Task**
    

```swift
Task.detached {
    // выполняется на глобальном executor
}
```

- Не привязан к текущему actor или MainActor.
    
- Полезен для фоновых вычислений.
    

3. **Task с результатом**
    

```swift
Task<Int, Error> {
    return 42
}
```

- Можно ожидать результат через `await task.value`.
    

---

## 3. Связь с async/await

- `Task` часто используется для вызова асинхронных функций из **синхронного контекста**.
    
- Внутри `Task` можно использовать `await`.
    

```swift
func fetchData() async -> String {
    "Data"
}

Task {
    let data = await fetchData()
    print(data)
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший Task

```swift
Task {
    print("Hello from async task")
}
```

- Код выполняется асинхронно на текущем executor.
    

---

### Пример 2. Task с результатом

```swift
Task<Int, Never> {
    return 5 + 3
}.value // await нужен для получения результата
```

```swift
Task {
    let result = await Task<Int, Never> { 5 + 3 }.value
    print(result) // 8
}
```

---

### Пример 3. Detached Task

```swift
Task.detached {
    print("Running in detached task: \(Thread.current)")
}
```

- Выполняется на глобальном executor.
    

---

### Пример 4. Task + cancellation

```swift
let task = Task {
    for i in 1...5 {
        try Task.checkCancellation()
        print("Step \(i)")
        try await Task.sleep(nanoseconds: 500_000_000)
    }
}

// Отмена задачи через 1 секунду
Task {
    try await Task.sleep(nanoseconds: 1_000_000_000)
    task.cancel()
}
```

- `Task.checkCancellation()` проверяет, отменена ли задача.
    

---

### Пример 5. Task + Sendable

```swift
struct DataModel: Sendable {
    let value: Int
}

Task.detached {
    let model = DataModel(value: 42)
    print(model.value) // безопасная передача в detached task
}
```

- `Task.detached` требует, чтобы все объекты были `Sendable`.
    

---

## 5. Особенности Task

1. **Каждая задача имеет свой executor**
    
    - Если не detached → наследует executor текущего контекста.
        
2. **Task.value**
    
    - Позволяет получать результат задачи
        
    - Обязательно `await`.
        
3. **Task.detached**
    
    - Полностью независимая задача
        
    - Используется для фоновых вычислений
        
4. **Task cancellation**
    
    - Задачи могут быть отменены через `task.cancel()`
        
    - Проверка через `Task.checkCancellation()`.
        
5. **Sendable requirement**
    
    - Detached task может принимать только `Sendable` данные.
        

---

## 6. Итог

- `Task` = асинхронная единица работы
    
- Выполняется на текущем executor или detached (глобальный executor)
    
- Может возвращать результат (`Task.value`)
    
- Поддерживает отмену и обработку ошибок
    
- Detached task требует Sendable данных
    

---
