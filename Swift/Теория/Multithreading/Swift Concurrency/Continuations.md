### 1. Что такое Continuation и зачем он нужен

**Continuation** — это **объект-мост**, который позволяет **превратить старый [[callback]]-based [[API]]** в современный `async/await`.

Проблема:
- До [[Swift]] Concurrency (2019–2021) почти все асинхронные API работали через **completion handler** (замыкание, вызываемое позже).
- [[Swift Concurrency]] не может «ждать» такие замыкания напрямую.

Решение:
- `withCheckedContinuation` / `withCheckedThrowingContinuation` **приостанавливает** текущую задачу (`suspend`)  
- передаёт continuation в замыкание  
- когда callback завершает работу → вызываем `continuation.resume(...)` → задача возобновляется с результатом

**Коротко и по-человечески**:
> Continuation = «я поставлю задачу на паузу, пока ты не скажешь мне результат через callback.  
> Как только callback сработает — я продолжу с твоим значением».

Это **самый мощный инструмент** для интеграции legacy-кода с async/await.

### 2. Два основных типа Continuation (2026)

| Тип                                      | Когда использовать                          | resume варианты                          | Проверка компилятора |
|------------------------------------------|---------------------------------------------|------------------------------------------|-----------------------|
| `CheckedContinuation<T, Never>`          | Нет ошибок (Result.success или просто T)   | `resume(returning: T)`                   | Да (один resume)      |
| `CheckedThrowingContinuation<T, Error>`  | Может быть ошибка                           | `resume(returning: T)` или `resume(throwing: Error)` | Да (один resume)      |
| `UnsafeContinuation<T, Never>`           | Низкоуровневый код, максимальная скорость   | То же                                    | Нет (unsafe)          |
| `UnsafeThrowingContinuation<T, Error>`   | То же + ошибки                              | То же                                    | Нет (unsafe)          |

**Рекомендация 2026**:
- 99% случаев → **Checked** / **CheckedThrowing**  
- `Unsafe` — только если ты **точно знаешь**, что делаешь (низкоуровневый код, производительность критично)

### 3. Самые популярные шаблоны 2026 года

#### Шаблон 1 — Классический Result → [[async]] [[throws]]

```swift
func oldFetch(completion: @escaping (Result<String, Error>) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        if Bool.random() {
            completion(.success("Успех"))
        } else {
            completion(.failure(NSError(domain: "Test", code: -1)))
        }
    }
}

func modernFetch() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        oldFetch { result in
            switch result {
            case .success(let value):
                continuation.resume(returning: value)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}

Task {
    do {
        let data = try await modernFetch()
        print(data)
    } catch {
        print("Ошибка:", error.localizedDescription)
    }
}
```

#### Шаблон 2 — Простой completion без ошибки

```swift
func oldTimer(completion: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
        completion()
    }
}

func modernTimer() async {
    await withCheckedContinuation { continuation in
        oldTimer {
            continuation.resume()
        }
    }
}

Task {
    print("Жду 2 секунды...")
    await modernTimer()
    print("Готово!")
}
```

#### Шаблон 3 — Отмена вложенной задачи (очень важно!)

```swift
func modernDownload(url: URL) async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        let task = URLSession.shared.dataTask(with: url) { data, _, error in
            if let error {
                continuation.resume(throwing: error)
            } else if let data {
                continuation.resume(returning: data)
            } else {
                continuation.resume(throwing: URLError(.badServerResponse))
            }
        }
        
        task.resume()
        
        // Важно: если задачу отменяют — отменяем URLSessionTask
        continuation.onCancellation {
            task.cancel()
        }
    }
}
```

#### Шаблон 4 — AsyncStream на базе Continuation (часто забывают)

```swift
func numbersStream() -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in 1...5 {
                continuation.yield(i)
                try? await Task.sleep(for: .seconds(1))
            }
            continuation.finish()
        }
    }
}

Task {
    for await num in numbersStream() {
        print(num)
    }
}
```

Здесь `AsyncStream` внутри использует Continuation-подобный механизм.

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                           | Последствия                                      | Как избежать                                         |
| ------------------------------------------------ | ------------------------------------------------ | ---------------------------------------------------- |
| Забыть вызвать resume                            | Задача висит вечно ([[deadlock]])                | Всегда resume ([[return]] / [[throw]])               |
| Вызвать resume дважды                            | [[Runtime crash]] (fatal error)                  | CheckedContinuation ловит в debug, но лучше один раз |
| Не отменять вложенные задачи                     | Задача продолжает работать после отмены [[Task]] | Использовать `continuation.onCancellation { ... }`   |
| Использовать UnsafeContinuation без причины      | Нет проверки → скрытые ошибки                    | Всегда Checked, если не критично                     |
| Забыть `try` в `withCheckedThrowingContinuation` | Ошибка компиляции                                | `try await` всегда                                   |
| Делать тяжёлую работу до resume                  | Блокирует весь актёр                             | Тяжёлое — в отдельный Task                           |

### 5. Continuation vs другие способы интеграции legacy (2026 сравнение)

| Механизм                          | Простота | Поддержка ошибок | Отмена вложенных задач | Проверка resume | Рекомендация 2026         |
| --------------------------------- | -------- | ---------------- | ---------------------- | --------------- | ------------------------- |
| `withCheckedThrowingContinuation` | Высокая  | Да               | Да (onCancellation)    | Да              | Основной выбор            |
| `withCheckedContinuation`         | Высокая  | Нет              | Да                     | Да              | Без ошибок                |
| `withUnsafeThrowingContinuation`  | Средняя  | Да               | Да                     | Нет             | Только производительность |
| AsyncStream / AsyncThrowingStream | Высокая  | Да               | Да                     | Да              | Для потоков данных        |
| [[Combine]] → AsyncPublisher      | Средняя  | Да               | Да                     | Да              | Legacy-проекты с Combine  |

**Вывод 2026**:
- `withCheckedThrowingContinuation` — **золотой стандарт** для обёртки любого callback-API  
- `AsyncStream` — когда нужен поток значений  
- `Unsafe` — только если ты **точно знаешь**, что делаешь (очень редко)

### 6. Лучшие практики 2026 года

- **Всегда используй Checked** (не Unsafe), если нет критической причины  
- **Вызывай resume ровно один раз** — лучше всего через `defer` или switch  
- **Обрабатывай отмену** — `continuation.onCancellation { ... }`  
- **Для UI** — финальное присваивание делай на `@MainActor`  
- **Тестирование** — проверяй, что при `Task.cancel()` вложенные операции останавливаются  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые resume  
- **Мониторинг** — Instruments → Swift Tasks — смотри suspension points и время ожидания

**Короткий девиз 2026**:
> «Continuation — это мост между старым callback-миром и новым async/await.  
> В 2026 году почти любой legacy API оборачивается именно через withCheckedThrowingContinuation.»
