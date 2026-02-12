## 1. Что такое Global Executor

**Global Executor** — это **исполнитель задач по умолчанию**, который [[Swift]] Concurrency использует для выполнения асинхронного кода, **если не указан другой executor**.

- Задачи выполняются **на системной пуле потоков**, т.е. на доступных фонах потоках.
    
- Отличие от `MainActor`:
    
    - MainActor → главный поток (UI)
        
    - Global Executor → фоновые потоки (background)
        

> Проще говоря, Global Executor = «любая свободная фоновая очередь, на которой можно выполнить задачу».

---

## 2. Когда используется

- При создании `Task { ... }` без `detached` или `MainActor.run`
    
- При асинхронных функциях без привязки к `actor` или `MainActor`
    
- Для фоновых вычислений и параллельных задач
    

---

## 3. Примеры

### Пример 1. Простая фоновая задача

```swift
Task {
    print("Выполняется на глобальном executor: \(Thread.current)")
}
```

- Выведет поток, который может отличаться от главного.
    

---

### Пример 2. Сравнение с MainActor

```swift
Task {
    print("Global Executor:", Thread.current)
}

Task { @MainActor in
    print("MainActor Executor:", Thread.current)
}
```

✅ Первый блок выполняется на фоновом потоке, второй — на главном потоке.

---

### Пример 3. Фоновые вычисления с результатом

```swift
func heavyCalculation() async -> Int {
    (1...1_000_000).reduce(0, +)
}

Task {
    let result = await heavyCalculation() // выполняется на Global Executor
    print("Сумма:", result)
}
```

---

### Пример 4. Параллельные фоновые задачи через [[async let]]

```swift
func taskA() async -> String { "A" }
func taskB() async -> String { "B" }

Task {
    async let a = taskA()
    async let b = taskB()
    
    let result = await "\(a)\(b)" // обе выполняются на Global Executor параллельно
    print(result) // AB
}
```

---

### Пример 5. Global Executor + [[Continuations]]

```swift
func oldAPICall(completion: @escaping (String) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        completion("Ответ старого API")
    }
}

func newAsyncCall() async -> String {
    await withCheckedContinuation { continuation in
        oldAPICall { result in
            continuation.resume(returning: result)
        }
    }
}

Task {
    let data = await newAsyncCall() // выполняется на Global Executor
    print(data)
}
```

---

## 4. Особенности

1. **Не гарантирует конкретный поток**
    
    - Задача может перейти на любой свободный поток пула.
        
2. **Используется по умолчанию**
    
    - Если не указать `@MainActor` или `Task.detached`, то задача запускается на Global Executor.
        
3. **Экономия ресурсов**
    
    - Swift Concurrency управляет пулом потоков автоматически.
        
4. **Можно переключать на другой executor**
    
    - Через `MainActor.run { ... }`
        
    - Или использовать `Task.detached` для полного отделения от текущего [[Executor]].
        

---

## 5. Итог

- Global Executor = **фоновые потоки по умолчанию**
    
- Задачи выполняются асинхронно без привязки к UI
    
- Используется для любых фоновых вычислений и async вызовов
    
- Отличие от MainActor и actor executor — **не привязан к конкретной изоляции и потоку**
    

---
