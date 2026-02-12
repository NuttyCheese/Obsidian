## 1. Что такое `await`

`await` — это **инструкция, которая приостанавливает выполнение текущей задачи**, пока не завершится асинхронная функция ([[async]]).

- Она говорит компилятору: «Подожди здесь результат, но не блокируй поток».
    
- [[Swift]] Concurrency автоматически позволяет другим задачам выполняться во время ожидания.
    

---

## 2. Синтаксис

```swift
let result = await asyncFunction()
```

- `asyncFunction()` — функция, помеченная `async`.
    
- `await` нужен **только для вызова `async` функций**.
    

---

## 3. Простые примеры

### Пример 1. Вызов async функции

```swift
func fetchData() async -> String {
    "Данные загружены"
}

Task {
    let result = await fetchData()
    print(result)
}
```

---

### Пример 2. Последовательный вызов

```swift
func fetchUser() async -> String { "Иван" }
func fetchOrders() async -> String { "Заказы" }

Task {
    let user = await fetchUser()      // ждём сначала пользователя
    let orders = await fetchOrders()  // затем заказы
    print(user, orders)
}
```

---

### Пример 3. Параллельный вызов с [[async let]]

```swift
Task {
    async let user = fetchUser()
    async let orders = fetchOrders()
    
    let u = await user
    let o = await orders
    
    print(u, o)
}
```

✅ `async let` позволяет выполнять функции **параллельно**, а `await` ждёт их результат.

---

### Пример 4. Работа с [[actor]]

```swift
actor Counter {
    var value = 0
    func increment() { value += 1 }
    func getValue() -> Int { value }
}

let counter = Counter()

Task {
    await counter.increment()        // доступ к actor через await
    let v = await counter.getValue()
    print(v)
}
```

---

### Пример 5. Асинхронный таймер

```swift
Task {
    for i in 1...5 {
        try? await Task.sleep(nanoseconds: 1_000_000_000) // ждём 1 секунду
        print("Тик \(i)")
    }
}
```

✅ `await` позволяет «приостановить» выполнение без блокировки потока.

---

## 4. Особенности `await`

1. **Нельзя использовать вне async контекста**
    

```swift
let result = await fetchData() // ❌ Ошибка, если не внутри async или Task
```

- Нужно обернуть в [[Task]] {} или вызвать из `async` функции.
    

2. **Сочетается с [[throws]]**
    

```swift
func fetchData() async throws -> String { "Данные" }

Task {
    do {
        let result = try await fetchData()
        print(result)
    } catch {
        print("Ошибка:", error)
    }
}
```

3. **Не блокирует поток**
    

- В отличие от `DispatchQueue.main.sync`, `await` приостанавливает задачу, а не поток.
    

---

## 5. Итог

- `await` = «подождать результат асинхронной функции».
    
- Используется внутри `async` функций или `Task`.
    
- Работает с `async let`, actor методами, таймерами, сетевыми вызовами.
    
- Обеспечивает безопасный, неблокирующий параллелизм.
    

---
