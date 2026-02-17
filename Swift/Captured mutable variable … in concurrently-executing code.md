Это предупреждение указывает, что **изменяемая (mutable) переменная была захвачена замыканием, которое выполняется параллельно или асинхронно**.

- [[Swift]] предупреждает, что это может привести к **race condition** — когда несколько потоков одновременно читают/пишут одну и ту же переменную, что может вызвать непредсказуемое поведение или краш.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: захват переменной в `DispatchQueue`**

```swift
var counter = 0

DispatchQueue.global().async {
    counter += 1  // ⚠️ Captured mutable variable 'counter' in concurrently-executing code
}
DispatchQueue.global().async {
    counter += 1
}
```

- Здесь две асинхронные задачи пытаются изменить `counter` одновременно → race condition.
    

---

**Пример 2: захват переменной в замыкании операции**

```swift
var array = [Int]()

let queue = OperationQueue()

for i in 0..<10 {
    queue.addOperation {
        array.append(i) // ⚠️ Captured mutable variable 'array' in concurrently-executing code
    }
}
```

- Несколько операций одновременно пишут в один массив → предупреждение компилятора.
    

---

### Как исправить

#### 1️⃣ Использовать [[DispatchQueue]] с синхронизацией

```swift
var counter = 0
let lockQueue = DispatchQueue(label: "counter.lock.queue")

DispatchQueue.global().async {
    lockQueue.sync {
        counter += 1
    }
}
DispatchQueue.global().async {
    lockQueue.sync {
        counter += 1
    }
}
```

- Теперь доступ к `counter` синхронизирован, race condition нет.
    

---

#### 2️⃣ Использовать атомарные структуры или `@Sendable` замыкания

```swift
let atomicCounter = ManagedAtomic<Int>(0)

DispatchQueue.global().async {
    atomicCounter.wrappingIncrement(ordering: .relaxed)
}
```

- `ManagedAtomic` (из [[Swift]] Atomics) безопасно работает с конкурентным доступом.
    

---

#### 3️⃣ Копировать значение вместо захвата mutable переменной

```swift
var value = 0
let localValue = value

DispatchQueue.global().async {
    print(localValue) // безопасно
}
```

- Захватываем **иммутабельную копию**, а не изменяемую переменную.
    

---

### Резюме

- Внимание: mutable переменные → [[race condition]] при конкурентном выполнении.
    
- Решения: синхронизация ([[NSLock]], DispatchQueue), атомарные структуры, копирование значений.
    
- Предупреждение компилятора помогает предотвратить потенциальные краши и непредсказуемое поведение.
    

---
