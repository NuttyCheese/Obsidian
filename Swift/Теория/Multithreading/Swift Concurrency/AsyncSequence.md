## 1. Что такое `AsyncSequence`

`AsyncSequence` — это **асинхронная последовательность** (аналог обычного [[Swift/Теория/Swift/Standart Library/Sequence]]), которая генерирует элементы **постепенно, с возможной задержкой**, и их можно обрабатывать с помощью [[for-await]].

- Используется, когда данные приходят не сразу (например, поток событий, загрузка данных из сети, таймеры).
    
- Каждый элемент может быть получен **асинхронно**, без блокировки текущего потока.
    

Пример аналогии: обычный массив — все элементы уже есть. AsyncSequence — элементы появляются постепенно.

---

## 2. Основной синтаксис

```swift
for await element in asyncSequence {
    // обработка каждого элемента
}
```

- `for await` — ключевое отличие от обычного `for`.
    
- Любой `AsyncSequence` должен реализовать метод `makeAsyncIterator() -> AsyncIterator`, который возвращает `AsyncIteratorProtocol`.
    

---

## 3. Простейший пример

### Пример 1. AsyncSequence с числовым диапазоном

```swift
struct AsyncNumbers: AsyncSequence {
    typealias Element = Int
    
    let count: Int
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var current = 1
        let max: Int
        
        mutating func next() async -> Int? {
            guard current <= max else { return nil }
            let value = current
            current += 1
            try? await Task.sleep(nanoseconds: 300_000_000) // имитация задержки
            return value
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        AsyncIterator(current: 1, max: count)
    }
}

// Использование
Task {
    let numbers = AsyncNumbers(count: 5)
    for await num in numbers {
        print(num)
    }
}
```

✅ Выведет числа 1..5 с задержкой ~0.3 сек каждое.

---

## 4. Пример 2. AsyncSequence с сетью

```swift
struct URLLines: AsyncSequence {
    typealias Element = String
    let url: URL
    
    struct AsyncIterator: AsyncIteratorProtocol {
        var lines: [String]
        var index = 0
        
        mutating func next() async -> String? {
            guard index < lines.count else { return nil }
            let line = lines[index]
            index += 1
            try? await Task.sleep(nanoseconds: 200_000_000)
            return line
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        // Имитация загрузки текста с сайта
        AsyncIterator(lines: ["line1", "line2", "line3"])
    }
}

// Использование
Task {
    let sequence = URLLines(url: URL(string: "https://example.com")!)
    for await line in sequence {
        print(line)
    }
}
```

---

## 5. Пример 3. AsyncStream

[[Swift]] предоставляет готовый тип **`AsyncStream`** для создания AsyncSequence из [[callback]] или событий:

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

✅ Очень удобно для событий, таймеров и потоков данных.

---

## 6. Пример 4. AsyncThrowingStream

Если поток может бросать ошибку:

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
        print("Ошибка в потоке:", error)
    }
}
```

---

## 7. Пример 5. Таймер как AsyncSequence

```swift
let timerStream = AsyncStream<Date> { continuation in
    Task {
        for _ in 1...5 {
            continuation.yield(Date())
            try? await Task.sleep(nanoseconds: 1_000_000_000) // 1 секунда
        }
        continuation.finish()
    }
}

Task {
    for await date in timerStream {
        print("Текущее время:", date)
    }
}
```

✅ Отлично подходит для обновлений UI или событий в приложении.

---

## 8. Итог

- `AsyncSequence` = последовательность элементов, которые приходят асинхронно.
    
- Используется с `for await` для безопасного получения данных.
    
- Можно создавать свой AsyncSequence через `AsyncIteratorProtocol` или использовать готовые типы (`AsyncStream`, `AsyncThrowingStream`).
    
- Идеально подходит для **сетевых потоков, таймеров, событий UI, actor сообщений**.
    

---
