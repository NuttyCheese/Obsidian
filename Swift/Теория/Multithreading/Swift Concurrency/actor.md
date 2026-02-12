## 1. Что такое `actor`

👉 `actor` — это специальный тип (как [[class]] или [[struct]]), который гарантирует **потокобезопасный доступ к своим данным**.  
Swift автоматически следит, чтобы доступ к состоянию `actor` был **только изнутри очереди актёра**.

Можно сказать:

- `struct` — [[Value Type]] (копируется)
    
- `class` — [[Reference Type]] (ссылочный, не потокобезопасный)
    
- `actor` — reference type **с автоматической защитой от гонок данных**
    

---

## 2. Зачем нужен `actor`

Проблема: если два потока меняют одно и то же свойство у [[class]], может случиться **гонка данных**.  
Решение: `actor` гарантирует, что в каждый момент времени только один поток работает с его внутренними данными.

---

## 3. Синтаксис

```swift
actor MyActor {
    var counter = 0

    func increment() {
        counter += 1
    }
}
```

---

## 4. Как обращаться к `actor`

❌ Нельзя просто вызвать метод:

```swift
let a = MyActor()
a.increment() // Ошибка: 'async' call in a function that does not support concurrency
```

✅ Нужно через `await`:

```swift
let a = MyActor()
Task {
    await a.increment()
}
```

---

## 5. Примеры от простого к сложному

### Пример 1. Простая инкапсуляция состояния

```swift
actor Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        value
    }
}

let counter = Counter()

Task {
    await counter.increment()
    print(await counter.getValue()) // 1
}
```

👉 Все операции защищены от гонок.

---

### Пример 2. Несколько задач одновременно

```swift
actor Logger {
    private var messages: [String] = []
    
    func log(_ message: String) {
        messages.append(message)
    }
    
    func getAll() -> [String] {
        messages
    }
}

let logger = Logger()

Task {
    await logger.log("Первое сообщение")
}

Task {
    await logger.log("Второе сообщение")
}

Task {
    print(await logger.getAll()) // ["Первое сообщение", "Второе сообщение"]
}
```

👉 Даже при одновременных вызовах данные не повреждаются.

---

### Пример 3. Изоляция снаружи

```swift
actor BankAccount {
    private var balance: Int = 0
    
    func deposit(_ amount: Int) {
        balance += amount
    }
    
    func withdraw(_ amount: Int) -> Bool {
        if balance >= amount {
            balance -= amount
            return true
        }
        return false
    }
    
    func getBalance() -> Int {
        balance
    }
}

let account = BankAccount()

Task {
    await account.deposit(100)
    print(await account.withdraw(50)) // true
    print(await account.getBalance()) // 50
}
```

👉 Всё изменяется атомарно.

---

### Пример 4. Взаимодействие нескольких `actor`

```swift
actor Wallet {
    private var coins: Int = 0
    
    func add(_ amount: Int) { coins += amount }
    func take(_ amount: Int) -> Bool {
        guard coins >= amount else { return false }
        coins -= amount
        return true
    }
}

actor Game {
    let wallet = Wallet()
    
    func play() async {
        await wallet.add(10)
        let success = await wallet.take(5)
        print(success ? "Успех!" : "Недостаточно монет")
    }
}

let game = Game()
Task {
    await game.play()
}
```

👉 `actor` может содержать другой `actor`, и всё работает синхронно через `await`.

---

### Пример 5. Nonisolated методы

Иногда нужно, чтобы метод был **доступен без `await`** (не требовал изоляции).  
Для этого есть `nonisolated`.

```swift
actor UserSession {
    private var token: String = "12345"
    
    func getToken() -> String {
        token
    }
    
    nonisolated func appVersion() -> String {
        "1.0.0"
    }
}

let session = UserSession()

Task {
    print(await session.getToken()) // требует await
    print(session.appVersion())     // доступно сразу
}
```

---

### Пример 6. Сравнение с классом

```swift
final class UnsafeCounter {
    var value = 0
    func increment() { value += 1 }
}

let unsafeCounter = UnsafeCounter()

DispatchQueue.concurrentPerform(iterations: 1000) { _ in
    unsafeCounter.increment()
}

print(unsafeCounter.value) // ❌ может быть не 1000
```

А с `actor`:

```swift
actor SafeCounter {
    var value = 0
    func increment() { value += 1 }
}

let safeCounter = SafeCounter()

Task {
    await withTaskGroup(of: Void.self) { group in
        for _ in 0..<1000 {
            group.addTask {
                await safeCounter.increment()
            }
        }
    }
    print(await safeCounter.value) // ✅ всегда 1000
}
```

---

## 6. Итог

- `actor` = потокобезопасный `class`
    
- Все свойства и методы по умолчанию **изолированы** (нужен [[await]] для доступа)
    
- Гарантирует отсутствие гонок данных
    
- Можно использовать `nonisolated` для методов без изоляции
    

---
