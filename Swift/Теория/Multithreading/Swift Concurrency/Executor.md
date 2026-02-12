## 1. Что такое `Executor`

**Executor** — это объект, который определяет **контекст выполнения асинхронного кода**.

- [[Swift]] Concurrency использует его для **гарантии изоляции данных и потокобезопасности**.
    
- Каждый [[actor]] имеет свой **executor**, и все его методы выполняются **внутри этого executor**.
    
- Executor отвечает за:
    
    - на каком потоке или очереди выполняется код,
        
    - порядок выполнения задач,
        
    - соблюдение изоляции акторов.
        

> Можно сказать: Executor — это «место, где [[Swift]] Concurrency запускает задачи».

---

## 2. Основные виды Executor

1. **MainActor executor**
    
    - Все задачи выполняются на **главном потоке** (UI-поток).
        
    - Используется для обновления UI.
        
2. **Actor executor**
    
    - Каждый `actor` имеет свой executor.
        
    - Гарантирует последовательный и безопасный доступ к состоянию actor.
        
3. **Detached executor (Task.detached)**
    
    - Задачи выполняются на **любой доступной очереди**, без привязки к actor или MainActor.
        
    - Используется для фоновых вычислений.
        

---

## 3. Связь с async/await

- Когда мы вызываем `await actor.method()` — код фактически **переходит на executor актера**, где выполняется метод.
    
- `MainActor.run { ... }` — выполняет код на главном потоке.
    
- `Task.detached` — создаёт задачу **без executor**, можно выбрать свой.
    

---

## 4. Примеры от простого к сложному

### Пример 1. MainActor executor

```swift
@MainActor
func updateUI() {
    print("Выполняется на главном потоке")
}

Task {
    await updateUI() // переключаемся на MainActor executor
}
```

---

### Пример 2. Actor executor

```swift
actor Counter {
    var value = 0
    func increment() {
        value += 1
        print("Executor актера: \(value)")
    }
}

let counter = Counter()

Task {
    await counter.increment() // выполняется в executor актера
}
```

---

### Пример 3. Detached executor

```swift
Task.detached {
    print("Выполняется на любом потоке, без привязки к actor")
}
```

---

### Пример 4. Комбинация MainActor и Detached

```swift
actor Logger {
    func log(_ message: String) {
        print("Logger executor:", message)
    }
}

let logger = Logger()

Task.detached {
    await logger.log("Сообщение с detached executor") // переключается на executor actor
    await MainActor.run {
        print("Обновляем UI на главном потоке")
    }
}
```

---

### Пример 5. Явное переключение executor

```swift
Task {
    print("Текущий поток до switch:", Thread.current)
    
    await MainActor.run {
        print("Выполняется на главном потоке:", Thread.current)
    }
    
    await Task.detached {
        print("Выполняется на detached executor:", Thread.current)
    }.value
}
```

---

## 5. Особенности

1. **Executor определяет поток и изоляцию**
    
    - Actor = один executor → гарантированная последовательность доступа.
        
    - MainActor = главный поток.
        
2. **Task.detached = без executor**
    
    - Задача может выполняться на любом потоке.
        
    - Не имеет изоляции actor.
        
3. **Слияние с async/await**
    
    - При вызове `await` Swift автоматически переключает код на нужный executor.
        
    - Не нужно вручную указывать очереди ([[DispatchQueue]]).
        
4. **Проверка компилятора**
    
    - [[Swift]] проверяет, что изоляция actor соблюдается при вызове методов через [[await]].
        

---

## 6. Итог

- Executor = «контекст выполнения» асинхронной задачи.
    
- Виды:
    
    1. **MainActor** → главный поток
        
    2. **Actor** → последовательный и безопасный доступ к actor
        
    3. **Detached executor** → фоновые задачи, без привязки
        
- `await` автоматически переключает код на executor цели.
    
- Позволяет писать безопасный многопоточный код без блокировок и гонок данных.
    

---
