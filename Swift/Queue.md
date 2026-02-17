**`Queue`** — структура или механизм для **упорядоченного хранения и обработки элементов**.  
В контексте [[iOS]] и [[Swift]] чаще всего встречаются:

1. **[[DispatchQueue]]** — часть **[[GCD]]**, используется для **асинхронного выполнения кода на разных потоках**.
    
2. **OperationQueue** — часть **[[Foundation]]**, используется для **управления очередями операций с возможностью приоритетов и зависимостей**.
    

Относится к **Concurrency / Multithreading**.

---

## 🔹 Примеры кода

### 1. Создание Serial DispatchQueue

```swift
let serialQueue = DispatchQueue(label: "com.example.serialQueue")

serialQueue.async {
    print("Задача 1")
}

serialQueue.async {
    print("Задача 2")
}
```

---

### 2. Создание Concurrent DispatchQueue

```swift
let concurrentQueue = DispatchQueue(label: "com.example.concurrentQueue", attributes: .concurrent)

concurrentQueue.async { print("Задача A") }
concurrentQueue.async { print("Задача B") }
```

---

### 3. Использование main queue

```swift
DispatchQueue.main.async {
    print("Выполняется на главном потоке (UI обновления)")
}
```

---

### 4. Delayed execution

```swift
DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
    print("Выполнится через 2 секунды")
}
```

---

### 5. Создание OperationQueue

```swift
let operationQueue = OperationQueue()
operationQueue.maxConcurrentOperationCount = 2

operationQueue.addOperation {
    print("Операция 1")
}

operationQueue.addOperation {
    print("Операция 2")
}
```

---

### 6. OperationQueue с зависимостями

```swift
let op1 = BlockOperation { print("Операция 1") }
let op2 = BlockOperation { print("Операция 2") }
op2.addDependency(op1)

let queue = OperationQueue()
queue.addOperations([op1, op2], waitUntilFinished: false)
```
