## 1. Что такое `for await`

`for await` — это цикл, который позволяет **итерировать элементы `AsyncSequence` по мере их появления**.

- Аналог обычного `for` для массивов, но для **асинхронного потока данных**.
    
- Каждый элемент может приходить с задержкой, и `for await` автоматически **приостанавливает выполнение** до появления следующего элемента.
    

---

## 2. Синтаксис

```swift
for await element in asyncSequence {
    // обработка элемента
}
```

- `asyncSequence` должен соответствовать протоколу `AsyncSequence`.
    
- Цикл сам вызывает `await` на каждом элементе.
    

---

## 3. Простейший пример

```swift
struct AsyncNumbers: AsyncSequence {
    typealias Element = Int
    let max: Int
    
    struct Iterator: AsyncIteratorProtocol {
        var current = 1
        let max: Int
        
        mutating func next() async -> Int? {
            guard current <= max else { return nil }
            let value = current
            current += 1
            try? await Task.sleep(nanoseconds: 200_000_000)
            return value
        }
    }
    
    func makeAsyncIterator() -> Iterator {
        Iterator(current: 1, max: max)
    }
}

Task {
    let numbers = AsyncNumbers(max: 5)
    for await num in numbers {
        print(num)
    }
}
```

✅ Выведет числа 1..5 с задержкой ~0.2 сек.

---

## 4. AsyncStream + for await

`AsyncStream` — встроенный тип для создания асинхронной последовательности из callback или событий.

```swift
let stream = AsyncStream<Int> { continuation in
    Task {
        for i in 1...5 {
            continuation.yield(i)
            try? await Task.sleep(nanoseconds: 300_000_000)
        }
        continuation.finish()
    }
}

Task {
    for await num in stream {
        print(num)
    }
}
```

---

## 5. AsyncThrowingStream + for try await

Если поток может бросать ошибки:

```swift
let throwingStream = AsyncThrowingStream<Int, Error> { continuation in
    Task {
        for i in 1...5 {
            if i == 3 {
                continuation.finish(throwing: NSError(domain: "Test", code: 1))
                return
            }
            continuation.yield(i)
            try? await Task.sleep(nanoseconds: 200_000_000)
        }
        continuation.finish()
    }
}

Task {
    do {
        for try await num in throwingStream {
            print(num)
        }
    } catch {
        print("Ошибка:", error)
    }
}
```

---

## 6. Использование с [[actor]]

```swift
actor Logger {
    private var messages: [String] = []

    func log(_ message: String) {
        messages.append(message)
    }

    func stream() -> AsyncStream<String> {
        AsyncStream { continuation in
            Task {
                for msg in await messages {
                    continuation.yield(msg)
                }
                continuation.finish()
            }
        }
    }
}

let logger = Logger()

Task {
    await logger.log("Сообщение 1")
    await logger.log("Сообщение 2")

    for await msg in await logger.stream() {
        print(msg)
    }
}
```

---

## 7. Особенности `for await`

1. **Работает только с `AsyncSequence`**
    
2. **Приостанавливает выполнение до следующего элемента**
    
3. Можно комбинировать с [[try]] если последовательность может выбросить ошибку
    
4. Отлично подходит для **событий, потоков данных, таймеров, сетевых стримов**
    

---

## 8. Итог

- `for await` = асинхронный цикл для элементов, которые появляются постепенно
    
- Используется с `AsyncSequence`, `AsyncStream`, `AsyncThrowingStream`
    
- Работает с actor и другими async [[API]]
    
- Автоматически управляет ожиданием элементов, без блокировки потоков
    

---
