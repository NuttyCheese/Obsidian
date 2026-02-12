## 1. Что такое `TaskGroup`

**`TaskGroup`** — это **контейнер для параллельных задач (child tasks)**, который позволяет:

- Запускать несколько задач одновременно
    
- Собрать их результаты
    
- Дождаться завершения всех задач
    

> Проще говоря: TaskGroup = «параллельный букет задач с синхронным ожиданием их завершения».

---

## 2. Основные виды TaskGroup

1. **TaskGroup** — для задач, которые возвращают значение:
    

```swift
func someFunction() async -> Int {
    1
}
```

```swift
await withTaskGroup(of: Int.self) { group in
    // добавляем задачи
}
```

2. **TaskGroup** — для задач, которые ничего не возвращают.
    

---

## 3. Основной синтаксис

```swift
await withTaskGroup(of: ReturnType.self) { group in
    group.addTask {
        // первая child task
    }
    
    group.addTask {
        // вторая child task
    }
    
    for await result in group {
        // обработка результатов по мере завершения задач
    }
}
```

- `addTask { ... }` — добавление дочерней задачи в группу
    
- `for await result in group` — перебор результатов по мере готовности
    
- `await withTaskGroup` — завершает работу после окончания всех дочерних задач
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший TaskGroup

```swift
Task {
    await withTaskGroup(of: Int.self) { group in
        group.addTask { 1 }
        group.addTask { 2 }
        for await value in group {
            print(value) // 1, 2 (порядок не гарантирован)
        }
    }
}
```

---

### Пример 2. Суммирование результатов

```swift
Task {
    let sum = await withTaskGroup(of: Int.self) { group in
        for i in 1...5 {
            group.addTask { i * 2 }
        }
        
        var total = 0
        for await value in group {
            total += value
        }
        return total
    }
    print("Total:", sum) // 30
}
```

---

### Пример 3. TaskGroup + [[async let]] (альтернатива)

```swift
Task {
    async let a = Task { 10 }.value
    async let b = Task { 20 }.value
    
    let sum = await a + b
    print(sum) // 30
}
```

- Отличие TaskGroup: можно динамически добавлять задачи во время выполнения.
    

---

### Пример 4. Обработка ошибок

```swift
enum MyError: Error { case fail }

Task {
    do {
        try await withThrowingTaskGroup(of: Int.self) { group in
            group.addTask { 1 }
            group.addTask { throw MyError.fail }
            
            for try await value in group {
                print(value)
            }
        }
    } catch {
        print("Ошибка:", error) // MyError.fail
    }
}
```

- `withThrowingTaskGroup` позволяет обрабатывать ошибки дочерних задач.
    

---

### Пример 5. TaskGroup с задержкой и порядком завершения

```swift
Task {
    await withTaskGroup(of: String.self) { group in
        group.addTask { 
            try? await Task.sleep(nanoseconds: 300_000_000)
            return "Task 1"
        }
        group.addTask { 
            try? await Task.sleep(nanoseconds: 100_000_000)
            return "Task 2"
        }
        group.addTask { 
            try? await Task.sleep(nanoseconds: 200_000_000)
            return "Task 3"
        }
        
        for await value in group {
            print(value) 
            // Output может быть: Task 2, Task 3, Task 1
        }
    }
}
```

- Результаты поступают **по мере готовности**, порядок не гарантирован.
    

---

## 5. Особенности TaskGroup

1. **Дочерние задачи наследуют executor родителя**
    
2. **Автоматическое ожидание всех задач** после выхода из блока `withTaskGroup`
    
3. **Можно динамически добавлять задачи** внутри блока
    
4. **Результаты и ошибки можно перебирать через [[for-await]]**
    
5. **Throwing и non-throwing варианты**:
    
    - `withTaskGroup(of:)` → без ошибок
        
    - `withThrowingTaskGroup(of:)` → с поддержкой async throw
        

---

## 6. Итог

- `TaskGroup` = контейнер для параллельных задач с автоматическим ожиданием
    
- Позволяет запускать динамическое количество дочерних задач
    
- `for await` перебирает результаты по мере завершения
    
- Существуют throwing и non-throwing варианты
    
- Отличие от `async let`: TaskGroup можно расширять динамически, [[async let]] фиксированное количество задач
    

---
