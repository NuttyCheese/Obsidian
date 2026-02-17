**`AsyncSequence`** — это **асинхронный аналог** протокола [[Sequence 1]].

Обычный `Sequence` даёт элементы **синхронно и сразу** (массив, диапазон, строка и т.д.).

`AsyncSequence` даёт элементы **постепенно**, с возможными **задержками** между ними, и их получение требует **асинхронного ожидания**.

**Главные отличия**:

| Характеристика              | Sequence (обычная)                  | AsyncSequence                              |
|-----------------------------|--------------------------------------|---------------------------------------------|
| Получение элементов         | Синхронно (`for ... in`)            | Асинхронно (`for await ... in`)            |
| Блокировка потока           | Может блокировать                   | Никогда не блокирует (suspendable)         |
| Источник данных             | Всё уже в памяти                    | Данные могут приходить постепенно (сеть, таймер, события) |
| Обработка ошибок            | Нет встроенной                      | Поддерживает `AsyncThrowingSequence`       |
| Основные сценарии           | Массивы, диапазоны, коллекции       | Потоки событий, загрузка по частям, таймеры, стримы из сети |

**Коротко и по-человечески**:
> AsyncSequence — это когда данные **не лежат все сразу**, а **приходят по одному** с паузами, и ты хочешь их обрабатывать по мере поступления, не блокируя поток.

### 2. Основной синтаксис и протоколы

```swift
for await элемент in asyncSequence {
    // обработка элемента
}
```

Чтобы тип стал `AsyncSequence`, он должен реализовать:

```swift
protocol AsyncSequence {
    associatedtype Element
    associatedtype AsyncIterator : AsyncIteratorProtocol where AsyncIterator.Element == Element
    
    func makeAsyncIterator() -> AsyncIterator
}
```

А итератор:

```swift
protocol AsyncIteratorProtocol {
    associatedtype Element
    mutating func next() async -> Element?
}
```

### 3. Готовые реализации в стандартной библиотеке и Foundation (2026)

| Тип / Источник                          | Что возвращает                          | Пример использования |
|-----------------------------------------|------------------------------------------|----------------------|
| `AsyncStream<Element>`                  | Произвольный поток (yield / finish)     | Таймеры, события, уведомления |
| `AsyncThrowingStream<Element, Error>`   | То же + может бросать ошибки            | Сетевые стримы, парсинг |
| `URLSession.AsyncBytes`                 | Байты из сети (по частям)               | `for try await byte in response.bytes { ... }` |
| `AsyncThrowingStream` из `lines`        | Строки из потока (например, из файла)   | `for try await line in file.lines { ... }` |
| `NotificationCenter.Notifications`      | Асинхронный поток уведомлений           | `for await notification in center.notifications(named: .didBecomeActive) { ... }` |
| `Timer.publish(every:on:in:)`           | Поток `Date` с заданным интервалом      | `for await date in timer { ... }` |
| `AsyncMapSequence`, `AsyncFilterSequence` и т.д. | Трансформеры над другими AsyncSequence   | `sequence.map { ... }`, `filter { ... }` |

### 4. Самые популярные шаблоны 2026 года

#### Шаблон 1 — Таймер как AsyncSequence (очень частый в UI)

```swift
let timer = AsyncStream<Date> { continuation in
    Task {
        var count = 0
        while count < 10 {
            continuation.yield(Date())
            try? await Task.sleep(for: .seconds(1))
            count += 1
        }
        continuation.finish()
    }
}

Task {
    for await date in timer {
        await MainActor.run {
            label.text = date.formatted()
        }
    }
}
```

#### Шаблон 2 — Поток строк из сети (реальный сценарий)

```swift
let url = URL(string: "https://example.com/stream.txt")!

Task {
    do {
        let (bytes, _) = try await URLSession.shared.bytes(from: url)
        
        for try await line in bytes.lines {
            print("Получена строка:", line)
            // например, парсинг JSONL, SSE, чат-стрим и т.д.
        }
    } catch {
        print("Ошибка стрима:", error)
    }
}
```

#### Шаблон 3 — AsyncThrowingStream + обработка ошибок

```swift
let stream = AsyncThrowingStream<Int, Error> { continuation in
    Task {
        for i in 1...10 {
            if i == 5 {
                continuation.finish(throwing: NSError(domain: "Test", code: -1))
                return
            }
            continuation.yield(i)
            try await Task.sleep(for: .milliseconds(300))
        }
        continuation.finish()
    }
}

Task {
    do {
        for try await number in stream {
            print("Получено:", number)
        }
    } catch {
        print("Поток прерван ошибкой:", error)
    }
}
```

#### Шаблон 4 — Комбинирование нескольких AsyncSequence

```swift
let timer = AsyncStream<Date> { ... }  // каждую секунду
let notifications = NotificationCenter.default.notifications(named: .didBecomeActive)

Task {
    for await (date, _) in merge(timer, notifications) {
        await MainActor.run {
            statusLabel.text = "Последнее событие: \(date.formatted())"
        }
    }
}
```

(Здесь используется `merge` из Swift Algorithms или кастомный `AsyncMergeSequence`.)

### 5. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `await` в `for await`                | Ошибка компиляции                        | Всегда `for await` |
| Использовать обычный `for` вместо `for await` | Ошибка компиляции                        | Компилятор напомнит |
| Долгая синхронная работа внутри `next()`    | Блокирует весь актёр / поток             | Всё тяжёлое — в отдельный `Task` |
| Не вызывать `continuation.finish()`         | Поток висит вечно                        | Всегда завершать (throwing или нормально) |
| Забыть обработку ошибок в `AsyncThrowingStream` | Не пойманная ошибка крашит задачу        | Всегда `do-try-await` |
| Слишком много элементов без backpressure    | Переполнение памяти                      | Использовать буферизацию или ограничение |

### 6. AsyncSequence vs другие механизмы потоков данных (2026 сравнение)

| Механизм                               | Параллелизм внутри потока | Обработка ошибок | Отмена | Backpressure | Рекомендация 2026 | Когда использовать     |
| -------------------------------------- | ------------------------- | ---------------- | ------ | ------------ | ----------------- | ---------------------- |
| `AsyncSequence` / `AsyncStream`        | Нет (один за раз)         | Да ([[throws]])  | Да     | Частично     | Основной выбор    | Потоки событий, стримы |
| `AsyncThrowingStream`                  | Нет                       | Да               | Да     | Частично     | Для ошибок        | Сетевые стримы, SSE    |
| [[Combine]] ([[Publisher]])            | Да (много подписчиков)    | Да               | Да     | Да           | Legacy / Rx       | Старые проекты         |
| [[NotificationCenter]] + AsyncSequence | Нет                       | Нет              | Да     | Нет          | Современный       | События приложения     |
| [[URLSession]] bytes / lines           | Нет                       | Да               | Да     | Да           | Стандартный       | Загрузка по частям     |

### 7. Лучшие практики 2026 года

- **Используй `AsyncStream`** для создания собственных потоков (таймеры, события, очереди)  
- **Для сети** — предпочитай `URLSession.bytes` / `lines` — это уже готовый `AsyncSequence`  
- **Для ошибок** — всегда `AsyncThrowingStream`  
- **Для UI** — финальное потребление делай на `@MainActor`  
- **Отмена** — используй `Task.cancel()` — `AsyncStream` сам завершится  
- **Backpressure** — если нужно ограничивать скорость — используй буферизацию или `AsyncChannel` (из swift-async-algorithms)  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые `await` и unsafe захваты  
- **Мониторинг** — Instruments → Swift Tasks + Swift Async Stream

**Короткий девиз 2026**:
> «AsyncSequence — это когда данные приходят не все сразу, а по одному, с паузами, и ты хочешь их обрабатывать по мере поступления.  
> В 2026 году это основной способ работы с потоками событий, сетевыми стримами и таймерами в Swift.»
