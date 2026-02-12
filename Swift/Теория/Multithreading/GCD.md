#multithreading #Swift 
## 📘 Определение

**GCD (Grand Central Dispatch)** — фреймворк Apple для **управления многопоточностью и асинхронным выполнением задач**.  
Позволяет распределять работу по **потокам (queues)** и выполнять задачи **параллельно или последовательно**, управляя приоритетами и синхронизацией.  
В [[Swift]] используется через **[[DispatchQueue]]**, **[[DispatchGroup]]**, **[[DispatchSemaphore]]** и другие [[API]].

---

## 🔹 Примеры кода

### 1. Простое выполнение задачи в фоновом потоке

```swift
import Foundation

DispatchQueue.global().async {
    print("Выполняем в фоне")
}
print("Главный поток")
```

---

### 2. Выполнение на главной очереди

```swift
DispatchQueue.main.async {
    print("Обновление UI на главной очереди")
}
```

---

### 3. Задержка выполнения

```swift
DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
    print("Выполнено через 2 секунды")
}
```

---

### 4. Создание последовательной и параллельной очереди

```swift
let serialQueue = DispatchQueue(label: "com.example.serial")
let concurrentQueue = DispatchQueue(label: "com.example.concurrent", attributes: .concurrent)

serialQueue.async { print("Последовательно 1") }
serialQueue.async { print("Последовательно 2") }

concurrentQueue.async { print("Параллельно 1") }
concurrentQueue.async { print("Параллельно 2") }
```

---

### 5. Использование DispatchGroup для синхронизации

```swift
let group = DispatchGroup()

group.enter()
DispatchQueue.global().async {
    print("Задача 1")
    group.leave()
}

group.enter()
DispatchQueue.global().async {
    print("Задача 2")
    group.leave()
}

group.notify(queue: .main) {
    print("Все задачи завершены")
}
```

---

### 6. Использование DispatchSemaphore для ограничения доступа

```swift
let semaphore = DispatchSemaphore(value: 1)

DispatchQueue.global().async {
    semaphore.wait()
    print("Начало критической секции 1")
    sleep(1)
    semaphore.signal()
}

DispatchQueue.global().async {
    semaphore.wait()
    print("Начало критической секции 2")
    semaphore.signal()
}
```
