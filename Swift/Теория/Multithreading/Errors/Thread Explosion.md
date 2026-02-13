### 1. Что такое Thread Explosion простыми словами

**Thread Explosion** — это ситуация, когда программа **создаёт слишком много потоков за короткое время**, и система **не успевает их обрабатывать или завершать**.

Каждый поток требует:
- выделения стека (обычно 512 КБ – 1 МБ на поток)  
- места в таблице потоков ядра  
- переключения контекста (context switch) при планировании

Когда потоков становится **сотни или тысячи** — начинаются проблемы:
- резкий рост потребления памяти  
- перегрузка планировщика задач ядра  
- сильное замедление всего приложения (даже UI может зависнуть)  
- в крайнем случае — **EXC_RESOURCE** или **jetsam** (убийство приложения системой)

**Коротко и по-человечески**:
> «Создал 10 000 потоков в цикле → приложение умерло от нехватки памяти и CPU, хотя задач было всего на 4 ядра.»

### 2. Самые частые сценарии Thread Explosion в [[iOS]]-приложениях 2026

| №   | Сценарий                                                      | Как проявляется                          | Частота |
| --- | ------------------------------------------------------------- | ---------------------------------------- | ------- |
| 1   | `Thread { ... }.start()` в цикле / рекурсии                   | Краш с EXC_RESOURCE / jetsam             | ★★★★★   |
| 2   | Рекурсивный вызов `Thread.detachNewThreadSelector`            | Приложение зависает, память растёт       | ★★★★☆   |
| 3   | `DispatchQueue.global().async` в глубокой рекурсии            | Пул [[GCD]] переполняется → замедление   | ★★★★☆   |
| 4   | `Task.detached` в цикле без ограничения                       | Сотни тысяч задач → высокая нагрузка CPU | ★★★★☆   |
| 5   | [[Таймер]] / [[CADisplayLink]] создаёт новый поток каждый тик | UI зависает, память утекает              | ★★★☆☆   |
| 6   | Сторонняя библиотека создаёт потоки без лимита                | Краш только при определённых сценариях   | ★★★☆☆   |

### 3. Классические примеры Thread Explosion

#### Пример 1 — Самый опасный (Thread в цикле)

```swift
for i in 0..<10_000 {
    Thread {
        print("Поток \(i) запущен")
        Thread.sleep(forTimeInterval: 60) // имитация долгой работы
    }.start()
}
```

**Результат**: приложение крашнется с `EXC_RESOURCE` или будет убито системой (jetsam) из-за нехватки памяти.

#### Пример 2 — Рекурсивный запуск потоков

```swift
func recursiveTask() {
    Thread {
        recursiveTask() // ← каждый вызов создаёт новый поток
    }.start()
}

recursiveTask() // → взрыв потоков за секунды
```

#### Пример 3 — Task.detached в цикле без ограничения

```swift
for url in 1000_urls {
    Task.detached {
        let image = try await downloadImage(url)
        await MainActor.run {
            images.append(image)
        }
    }
}
```

**Результат**: тысячи задач → высокая нагрузка CPU, задержки в UI, возможный jetsam.

### 4. Все современные способы защиты от Thread Explosion (2026)

| Способ                                                  | Защита от Thread Explosion | Скорость | Сложность | Рекомендация 2026          | Примечание                        |
| ------------------------------------------------------- | -------------------------- | -------- | --------- | -------------------------- | --------------------------------- |
| **[[DispatchQueue]]** (global/[[concurrent]] с лимитом) | ★★★★★                      | ★★★★★    | ★★☆☆☆     | Основной выбор для [[GCD]] | maxConcurrentOperationCount       |
| **[[OperationQueue]]**                                  | ★★★★★                      | ★★★★☆    | ★★★☆☆     | Legacy + сложные задачи    | maxConcurrentOperationCount       |
| **[[TaskGroup]] + ограничение concurrency**             | ★★★★★                      | ★★★★★    | ★★★★☆     | Асинхронные задачи         | Лучший способ в Swift Concurrency |
| **[[actor]]** + **[[Task]]**                            | ★★★★☆                      | ★★★★☆    | ★★★☆☆     | Состояние + задачи         | Не создаёт лишних потоков         |
| **[[@MainActor]] / глобальные очереди**                 | ★★★★☆                      | ★★★★☆    | ★★☆☆☆     | UI и сервисы               | Ограничивает количество потоков   |
| **[[DispatchWorkItem]] + cancellation**                 | ★★★★☆                      | ★★★★☆    | ★★★☆☆     | Отмена задач               | Для долгих операций               |
| **BGTaskScheduler / BackgroundTasks**                   | ★★★★☆                      | ★★★★☆    | ★★★★☆     | Фоновые задачи             | Системные ограничения             |

### 5. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### Вариант 1 — TaskGroup с ограничением concurrency

```swift
await withTaskGroup(of: Void.self) { group in
    for url in urls.prefix(8) {  // лимит 8 параллельных задач
        group.addTask(priority: .utility) {
            do {
                let image = try await downloadImage(url)
                await MainActor.run {
                    images.append(image)
                }
            } catch {
                print("Ошибка загрузки: \(error)")
            }
        }
    }
}
```

#### Вариант 2 — OperationQueue с лимитом

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4  // не больше 4 потоков одновременно
queue.qualityOfService = .utility

for url in urls {
    let operation = BlockOperation {
        // тяжёлая работа
        let image = downloadSync(url)
        DispatchQueue.main.async {
            images.append(image)
        }
    }
    queue.addOperation(operation)
}
```

#### Вариант 3 — actor + ограниченный пул задач

```swift
actor DownloadManager {
    private var activeDownloads = 0
    private let maxConcurrent = 4
    
    func download(_ url: URL) async throws -> Data {
        while activeDownloads >= maxConcurrent {
            try await Task.sleep(nanoseconds: 100_000_000) // backoff
        }
        
        activeDownloads += 1
        defer { activeDownloads -= 1 }
        
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}
```

### 6. Как найти Thread Explosion в 2026

| Инструмент / Способ               | Что ловит                              | Где включается                  | Эффективность |
|-----------------------------------|----------------------------------------|----------------------------------|---------------|
| **Instruments → Allocations**     | Взрывный рост памяти (потоки + стеки)  | Instruments → Allocations        | ★★★★★         |
| **Instruments → Threads**         | Сотни/тысячи потоков одновременно      | Instruments → Threads            | ★★★★★         |
| **Xcode Debugger → Threads**      | Список всех потоков в реальном времени | Debug navigator → Threads        | ★★★★☆         |
| **os_signpost + Console**         | Создание большого числа задач          | Console / os_log                 | ★★★★☆         |
| **Energy Impact / CPU Usage**     | Постоянная высокая загрузка CPU        | Instruments → Energy / CPU       | ★★★★☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Никогда** не создавайте `Thread { ... }.start()` в цикле или рекурсии  
- **Ограничивайте concurrency** — `TaskGroup`, `OperationQueue.maxConcurrentOperationCount`, `actor` с backoff  
- **Используйте правильные приоритеты** — `.userInteractive` и `.userInitiated` только для критических задач  
- **Фоновые задачи** — `.utility` и `.background`, не перегружайте их  
- **Swift 6 strict concurrency** — помогает косвенно, требуя Sendable и изоляцию  
- **Для тестов** — используйте `TaskGroup` с ограничением и проверяйте память/CPU  
- **Для legacy-кода** — постепенно заменяйте `Thread` на `Task` / `actor` / `OperationQueue`  
- **Мониторинг** — добавляйте os_signpost при создании задач и следите за Instruments

**Короткий девиз 2026**:
> «Thread Explosion — это когда ты случайно создал 10 000 потоков вместо 4.  
> В 2026 году ответ один: TaskGroup + ограничение concurrency + actor + правильные приоритеты.  
> Thread.start(), рекурсивные Task.detached и бесконечные циклы в high-priority — это путь к краху.»
