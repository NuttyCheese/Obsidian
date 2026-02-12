## 1. Что такое `async`

👉 `async` — это ключевое слово, которое помечает функцию (или метод) как **асинхронную**.  
Такая функция может **приостанавливаться** во время выполнения, чтобы подождать результат долгой операции (например, сети, чтения файла, анимации).

- Обычная функция работает «сразу и до конца».
    
- Асинхронная (`async`) может сказать: «подожду», отпустить поток, и потом продолжить с того же места.
    

---

## 2. Как использовать

- `async` ставится перед [[func]].
    
- Вызов такой функции требует `await`.
    

```swift
func fetchData() async -> String {
    return "Данные загружены"
}

Task {
    let result = await fetchData()
    print(result)
}
```

---

## 3. Взаимодействие с `await`

- `async` говорит: «эта функция может приостановиться».
    
- `await` говорит: «тут мы подождём результат асинхронной функции».
    

---

## 4. Примеры (от простого к сложному)

### Пример 1. Самая простая асинхронная функция

```swift
func sayHello() async {
    print("Привет 👋")
}

Task {
    await sayHello()
}
```

---

### Пример 2. Возвращаем значение

```swift
func getNumber() async -> Int {
    return 42
}

Task {
    let number = await getNumber()
    print(number) // 42
}
```

---

### Пример 3. Эмуляция задержки

Используем `Task.sleep` для имитации долгой операции (например, загрузки из сети).

```swift
func loadData() async -> String {
    try? await Task.sleep(nanoseconds: 1_000_000_000) // 1 секунда
    return "Данные получены"
}

Task {
    print("Начали загрузку…")
    let data = await loadData()
    print(data) // через 1 секунду
}
```

---

### Пример 4. Несколько асинхронных вызовов

```swift
func fetchUser() async -> String {
    "Пользователь: Иван"
}

func fetchOrders() async -> String {
    "Заказы: [#1, #2, #3]"
}

Task {
    let user = await fetchUser()
    let orders = await fetchOrders()
    print(user)
    print(orders)
}
```

👉 Выполняется последовательно.

---

### Пример 5. Параллельные вызовы через [[async let]]

```swift
func fetchUser() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000) // 0.5 сек
    return "Пользователь: Иван"
}

func fetchOrders() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000) // 0.5 сек
    return "Заказы: [#1, #2, #3]"
}

Task {
    async let user = fetchUser()
    async let orders = fetchOrders()
    
    // оба выполняются параллельно
    print(await user)
    print(await orders)
}
```

👉 Работает быстрее, чем последовательные `await`.

---

### Пример 6. Асинхронная функция с [[throws]]

```swift
enum NetworkError: Error {
    case failed
}

func fetchData() async throws -> String {
    let success = Bool.random()
    if success {
        return "Успешный ответ"
    } else {
        throw NetworkError.failed
    }
}

Task {
    do {
        let result = try await fetchData()
        print(result)
    } catch {
        print("Ошибка сети:", error)
    }
}
```

---

### Пример 7. Асинхронный [[API]] в [[iOS]]

```swift
func fetchImage() async throws -> UIImage {
    let url = URL(string: "https://picsum.photos/200")!
    let (data, _) = try await URLSession.shared.data(from: url)
    guard let image = UIImage(data: data) else {
        throw URLError(.badServerResponse)
    }
    return image
}

Task {
    do {
        let image = try await fetchImage()
        print("Картинка загружена: \(image)")
    } catch {
        print("Не удалось загрузить картинку:", error)
    }
}
```

---

## 5. Итог

- `async` → функция может приостановиться
    
- `await` → ждём результат этой функции
    
- Можно совмещать с `throws`, `async let`, `TaskGroup` и `actor`
    

---
