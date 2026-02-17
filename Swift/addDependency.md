**addDependency(_:)** — это метод класса [[Operation]], который позволяет создать **зависимость** между операциями:

- Операция A **добавляет зависимость** от операции B → A **не начнёт выполняться**, пока B **не завершится** (`isFinished == true`)

**Ключевые правила**:

- Зависимости работают **только внутри одной `OperationQueue`**  
- Зависимая операция (`A`) ждёт **всех** своих зависимостей  
- Зависимости **не отменяют** друг друга автоматически  
- Можно создавать **сложные цепочки** и **графы** зависимостей  
- Если операция отменена до запуска → она **не ждёт** зависимостей  
- Зависимости **не влияют** на [[queuePriority]] — приоритет внутри очереди отдельно

**Важнейший факт 2026 года**:
> `addDependency` — один из самых сильных аргументов в пользу [[OperationQueue]] в legacy-коде.  
> В [[Swift Concurrency]] **нет прямого аналога** сложных зависимостей — там используют [[TaskGroup]] + [[async let]] + ручную координацию или [[actor]].

### 2. Основные сценарии использования addDependency

| Сценарий                                      | Почему именно addDependency                  | Альтернатива в 2026 (рекомендуемая) |
|-----------------------------------------------|----------------------------------------------|--------------------------------------|
| Последовательная цепочка: загрузка → обработка → сохранение | Гарантированный порядок выполнения           | `async let` + `await` в одном Task   |
| Параллельная загрузка нескольких данных → одна финальная обработка | Финальная операция ждёт все загрузки         | `TaskGroup` + `await group.waitForAll()` |
| Миграция базы данных + загрузка конфига → запуск UI | UI запускается только после обеих задач      | `actor` + `Task` координация         |
| Legacy-код с зависимостями                    | Уже используется в старом коде               | Постепенная миграция на Concurrency  |

### 3. Самые популярные шаблоны использования в 2026 году

#### Шаблон 1 — Простая последовательная цепочка

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1  // для строгого порядка можно 1
queue.qualityOfService = .userInitiated

let fetchOp = BlockOperation {
    let data = fetchDataSync()
    // сохраняем в промежуточный результат
}

let processOp = BlockOperation {
    let processed = processDataSync()
    // сохраняем результат
}

let saveOp = BlockOperation {
    saveToDiskSync()
}

processOp.addDependency(fetchOp)
saveOp.addDependency(processOp)

queue.addOperations([fetchOp, processOp, saveOp], waitUntilFinished: false)

queue.addBarrierBlock {
    DispatchQueue.main.async {
        self.showSuccessMessage()
    }
}
```

#### Шаблон 2 — Параллельная загрузка → одна финальная операция

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4

var downloadOps: [Operation] = []

for url in urls {
    let op = BlockOperation {
        let image = downloadImageSync(url)
        // сохраняем в общий кэш (лучше через actor, но для примера)
        cacheImage(image, url: url)
    }
    downloadOps.append(op)
}

let finalizeOp = BlockOperation {
    // финальная обработка после всех загрузок
    updateUIAfterDownloads()
}

finalizeOp.addDependencies(downloadOps)

queue.addOperations(downloadOps + [finalizeOp], waitUntilFinished: false)
```

#### Шаблон 3 — Отмена всей цепочки

```swift
let fetchOp = BlockOperation { ... }
let processOp = BlockOperation { ... }
processOp.addDependency(fetchOp)

queue.addOperations([fetchOp, processOp], waitUntilFinished: false)

// Отмена при уходе с экрана
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    fetchOp.cancel()
    processOp.cancel()  // отменяет и зависимые
}
```

### 4. Типичные ошибки с addDependency в 2026

| Ошибка                                   | Последствия                         | Как избежать                                      |
| ---------------------------------------- | ----------------------------------- | ------------------------------------------------- |
| Забыть добавить зависимость              | Операция запускается раньше времени | Всегда явно вызывать `addDependency`              |
| Создать циклическую зависимость          | [[deadlock]] (очередь зависает)     | Проверять граф зависимостей                       |
| Добавлять зависимость после addOperation | Изменение игнорируется              | Устанавливать зависимости до добавления в очередь |
| Долгая зависимая операция без отмены     | Задержка после отмены               | Проверять `isCancelled` внутри операций           |
| Обновление UI напрямую из операций       | [[Main Thread Violation]]           | Всегда `@MainActor` / `DispatchQueue.main.async`  |

### 5. addDependency vs современные альтернативы (2026 сравнение)

| Механизм                  | Зависимости (сложные цепочки) | Отмена задач | Ограничение concurrency | Потокобезопасность | Рекомендация 2026 | Когда использовать |
|---------------------------|-------------------------------|--------------|--------------------------|---------------------|-------------------|---------------------|
| Operation.addDependency   | Да                            | Да           | Да (через queue)         | Ручная              | Legacy            | Зависимости, миграция |
| actor + Task              | Частично (ручная координация) | Да           | Да (через логику)        | Встроенная          | Основной выбор    | Изменяемое состояние |
| TaskGroup + await         | Нет (только параллель)        | Да           | Да (addTask лимит)       | Встроенная          | Параллельные задачи | Асинхронные загрузки |
| async let                 | Нет                           | Да           | Да (параллельно)         | Встроенная          | Простые сценарии   | 2–5 независимых задач |

**Вывод 2026**:
- **Новый код** → **TaskGroup** / **actor** + **async let**  
- **Сложные зависимости** → **OperationQueue + addDependency** (если не хотите переписывать)  
- Полная миграция → всё на Swift Concurrency (ручная координация через `actor` или `TaskGroup`)

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — OperationQueue + addDependency уже legacy  
- **addDependency** — используйте только для реальных зависимостей (не для простого порядка)  
- **maxConcurrentOperationCount** — всегда ограничивайте (4–8)  
- **isCancelled** — проверяйте в начале каждой операции  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `XCTestExpectation` + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных между зависимыми операциями

**Короткий девиз 2026**:
> «addDependency — это когда нужно сказать: «эта операция может начаться только после завершения этих».  
> В 2026 году это уже legacy-механизм.  
> Новый стандарт — TaskGroup + actor + async let + strict concurrency.  
> Если ты всё ещё используешь addDependency в новом коде — спроси себя: «А точно ли это нужно?»»
