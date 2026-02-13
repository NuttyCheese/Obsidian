Вот **полное, подробное и максимально насыщенное** руководство по конструкции **`for await`** в Swift — актуально на 2026 год (Swift 6+ и строгий режим конкурентности).

### 1. Что такое `for await` и зачем он нужен

**`for await`** — это **асинхронный аналог** обычного цикла `for-in`, предназначенный специально для работы с **`AsyncSequence`**.

Обычный цикл `for item in sequence` работает с синхронными последовательностями (массив, диапазон, строка и т.д.).

`for await item in asyncSequence` работает с **асинхронными последовательностями**, где:

- элементы появляются **не сразу**, а **по мере готовности**  
- между элементами могут быть **задержки** (сеть, таймер, события)  
- получение каждого элемента требует **асинхронного ожидания** (`await`)  
- цикл **приостанавливает** выполнение задачи до появления следующего элемента

**Коротко и по-человечески**:
> `for await` — это когда ты говоришь: «давай элементы по одному, как только они будут готовы, а я буду их обрабатывать по мере поступления, не блокируя поток».

Это один из самых мощных и часто используемых инструментов **структурированной конкурентности** в Swift.

### 2. Основные правила for await (2026)

| Правило                                   | Что происходит                                   | Пример ошибки / правильный код |
|-------------------------------------------|--------------------------------------------------|---------------------------------|
| Работает **только** с типами, реализующими `AsyncSequence` | Компилятор заставит                              | `for await x in [1,2,3]` → ошибка<br>`for await x in AsyncStream<Int> { ... }` → ок |
| **Обязательно** `await` в заголовке цикла | Без `await` — ошибка компиляции                  | `for item in stream` → ошибка<br>`for await item in stream` → ок |
| Цикл **приостанавливает** задачу до следующего элемента | Поток освобождается для других задач             | Нет блокировки даже при долгом ожидании |
| Если последовательность `AsyncThrowingSequence` — нужен `try await` | Ошибка компиляции без `try`                      | `for await x in stream` → ошибка<br>`for try await x in throwingStream` → ок |
| Цикл **завершается**, когда итератор возвращает `nil` | `next()` возвращает `nil` → выход из цикла       | `continuation.finish()` → цикл заканчивается |
| Можно **прервать** цикл через `break` / `return` / `throw` | Задача продолжает выполняться, но цикл останавливается | Нормально работает |

### 3. Самые популярные шаблоны for await 2026 года

#### Шаблон 1 — Таймер как поток (самый частый в UI)

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
            clockLabel.text = date.formatted(date: .omitted, time: .standard)
        }
    }
}
```

#### Шаблон 2 — Поток строк из сети (реальный сценарий)

```swift
let url = URL(string: "https://example.com/log.txt")!

Task {
    do {
        let (bytes, _) = try await URLSession.shared.bytes(from: url)
        
        for try await line in bytes.lines {
            print("Получена строка:", line)
            // парсинг логов, SSE, чат-стрим и т.д.
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
let timer = AsyncStream<Date> { ... }           // каждую секунду
let notifications = NotificationCenter.default.notifications(named: .didBecomeActive)

Task {
    for await event in merge(timer, notifications) {
        await MainActor.run {
            statusLabel.text = "Последнее событие: \(event)"
        }
    }
}
```

(Здесь `merge` — из пакета `swift-async-algorithms` или кастомная реализация.)

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Написать обычный `for` вместо `for await`   | Ошибка компиляции                        | Всегда `for await` для AsyncSequence |
| Забыть `try` в `for try await`              | Ошибка компиляции                        | Если последовательность throwing — `try await` |
| Делать тяжёлую синхронную работу внутри цикла | Блокирует весь актёр / поток             | Тяжёлое — в отдельный `Task` |
| Не вызывать `continuation.finish()`         | Цикл висит вечно                         | Всегда завершать поток |
| Забыть обработку ошибок                     | Не пойманная ошибка крашит задачу        | Всегда `do-try-await` |
| Слишком много элементов без backpressure    | Переполнение памяти                      | Использовать ограничение или буферизацию |

### 5. for await vs другие способы обработки потоков (2026 сравнение)

| Механизм                  | Тип данных            | Обработка ошибок | Отмена | Backpressure | Рекомендация 2026 | Когда использовать |
|---------------------------|-----------------------|-------------------|--------|--------------|-------------------|---------------------|
| `for await ... in AsyncSequence` | AsyncSequence         | Да (throws)       | Да     | Частично     | Основной выбор    | Все потоки данных |
| `AsyncStream` / `AsyncThrowingStream` | Произвольный поток    | Да                | Да     | Частично     | Для создания      | Таймеры, события, уведомления |
| `URLSession.bytes` / `lines` | Байты / строки из сети | Да                | Да     | Да           | Стандартный       | Загрузка по частям |
| Combine Publisher → AsyncPublisher | Publisher             | Да                | Да     | Да           | Legacy            | Старые проекты с Combine |
| NotificationCenter + AsyncSequence | Уведомления           | Нет               | Да     | Нет          | Современный       | События приложения |

### 6. Лучшие практики 2026 года

- **Используй `for await`** для всех асинхронных потоков данных  
- **Для создания потоков** — предпочитай `AsyncStream` / `AsyncThrowingStream`  
- **Для сети** — `URLSession.bytes` / `lines` — это уже готовый `AsyncSequence`  
- **Для UI** — финальное потребление делай на `@MainActor`  
- **Отмена** — используй `Task.cancel()` — поток завершится автоматически  
- **Backpressure** — если нужно ограничивать скорость — используй буферизацию или `AsyncChannel` (из swift-async-algorithms)  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые `await` и unsafe захваты  
- **Мониторинг** — Instruments → Swift Tasks + Swift Async Stream

**Короткий девиз 2026**:
> «for await — это когда ты говоришь: «давай мне элементы по одному, как только они будут готовы, а я буду их обрабатывать по мере поступления».  
> В 2026 году это основной способ работы с потоками событий, сетевыми стримами, таймерами и уведомлениями в Swift.»

Удачи с элегантными и безопасными асинхронными циклами в Swift! 🌊