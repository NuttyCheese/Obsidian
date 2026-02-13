### 1. Что такое Operation и зачем она нужна

**Operation** — это **абстрактный базовый класс** из фреймворка [[Foundation]], который представляет **одну независимую задачу** (синхронную или асинхронную), которую можно:

- выполнять в [[OperationQueue]]  
- отменять (`cancel()`)  
- делать зависимой от других операций ([[addDependency]])  
- отслеживать состояние (`isReady`, `isExecuting`, `isFinished`, [[isCancelled]])  
- задавать приоритет внутри очереди ([[queuePriority]])  
- получать уведомление о завершении ([[completionBlock]])

**Ключевые преимущества Operation (даже в 2026 году)**:

- **встроенные зависимости** между задачами  
- **отмена** (cancel) с проверкой внутри операции (`isCancelled`)  
- **контроль параллелизма** через `OperationQueue.maxConcurrentOperationCount`  
- **[[KVO]]-поддержка** состояний (можно наблюдать за isFinished и т.д.)  
- **[[QoS]]** и **queuePriority** для тонкой настройки внутри очереди

**Важнейший факт 2026**:
> `Operation` / `OperationQueue` — это **уже legacy-инструмент** (с 2014–2021 годов).  
> Apple **очень сильно рекомендует** переходить на **Swift Concurrency** (`Task`, `actor`, `TaskGroup`, `async let`).  
> Но `Operation` всё ещё активно используется:
> - в legacy-коде  
> - при сложных зависимостях  
> - в библиотеках, которые не мигрировали на async/await  
> - когда нужен KVO или очень тонкий контроль

### 2. Основные состояния и свойства Operation

| Свойство / Состояние | Тип                     | Что означает                                          | Когда меняется                          | Как часто проверять    |
| -------------------- | ----------------------- | ----------------------------------------------------- | --------------------------------------- | ---------------------- |
| `isReady`            | Bool                    | Операция готова к запуску (все зависимости завершены) | Автоматически                           | Редко                  |
| `isExecuting`        | Bool                    | Операция сейчас выполняется                           | Вручную (при [[isAsynchronous]] = true) | В кастомных операциях  |
| `isFinished`         | Bool                    | Операция завершена (успешно или отменена)             | Вручную (при isAsynchronous = true)     | В кастомных операциях  |
| `isCancelled`        | Bool                    | Операция была отменена (cancel())                     | Автоматически                           | Всегда проверять!      |
| `queuePriority`      | Operation.QueuePriority | Приоритет внутри очереди (.veryLow … .veryHigh)       | До добавления в очередь                 | Редко                  |
| `qualityOfService`   | QualityOfService        | QoS всей операции (наследуется от очереди)            | До добавления                           | Рекомендуется задавать |
| `completionBlock`    | (() -> Void)?           | Замыкание, вызывается после завершения/отмены         | Один раз в конце                        | Для финальных действий |
| `dependencies`       | [Operation]             | Список операций, от которых зависит эта               | До добавления                           | Для сложных цепочек    |

### 3. Самые популярные шаблоны использования в 2026 году

#### Шаблон 1 — Простая операция с отменой ([[BlockOperation]])

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4
queue.qualityOfService = .utility

let operation = BlockOperation { [weak self] in
    guard let self else { return }
    
    if operation.isCancelled { return }
    
    // Тяжёлая работа
    let data = fetchDataSync()
    
    if operation.isCancelled { return }
    
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}

operation.completionBlock = {
    print("Операция завершена или отменена")
}

queue.addOperation(operation)

// Отмена при уходе с экрана
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    operation.cancel()
}
```

#### Шаблон 2 — Кастомная асинхронная операция (isAsynchronous = true)

```swift
class AsyncNetworkOperation: Operation {
    override var isAsynchronous: Bool { true }
    
    private var _isExecuting: Bool = false {
        willSet { willChangeValue(forKey: "isExecuting") }
        didSet { didChangeValue(forKey: "isExecuting") }
    }
    
    override var isExecuting: Bool { _isExecuting }
    
    private var _isFinished: Bool = false {
        willSet { willChangeValue(forKey: "isFinished") }
        didSet { didChangeValue(forKey: "isFinished") }
    }
    
    override var isFinished: Bool { _isFinished }
    
    private let url: URL
    
    init(url: URL) {
        self.url = url
        super.init()
    }
    
    override func start() {
        if isCancelled {
            finish()
            return
        }
        
        _isExecuting = true
        
        URLSession.shared.dataTask(with: url) { [weak self] data, _, error in
            guard let self else { return }
            
            // обработка результата...
            
            self.finish()
        }.resume()
    }
    
    private func finish() {
        _isExecuting = false
        _isFinished = true
    }
}
```

#### Шаблон 3 — Зависимости между операциями

```swift
let queue = OperationQueue()

let fetchOp = BlockOperation { /* загрузка данных */ }
let processOp = BlockOperation { /* обработка */ }
let saveOp = BlockOperation { /* сохранение */ }

processOp.addDependency(fetchOp)
saveOp.addDependency(processOp)

queue.addOperations([fetchOp, processOp, saveOp], waitUntilFinished: false)

queue.addBarrierBlock {
    DispatchQueue.main.async {
        self.showSuccess()
    }
}
```

### 4. Типичные ошибки с Operation в 2026

| Ошибка                                                       | Последствия                             | Как избежать                                       |
| ------------------------------------------------------------ | --------------------------------------- | -------------------------------------------------- |
| Забыть проверять `isCancelled`                               | Операция продолжает работу после отмены | Проверять в начале и в циклах                      |
| Долгая работа без проверки отмены                            | Задержка после отмены                   | `if isCancelled { return }`                        |
| Обновление UI напрямую из операции                           | [[Main Thread Violation]]               | Всегда `DispatchQueue.main.async` или `@MainActor` |
| [[maxConcurrentOperationCount]] = -1 (без лимита)            | [[Thread Explosion]]                    | Устанавливать 4–8                                  |
| Неправильное управление isFinished при isAsynchronous = true | Операция «зависает» навсегда            | KVO + finish() в конце                             |
| Использование sync внутри операции                           | [[Deadlock]]                            | Только async                                       |

### 5. Operation vs современные альтернативы (2026 сравнение)

| Механизм                       | Зависимости | Отмена задач | Ограничение concurrency | Асинхронность по умолчанию | Рекомендация 2026   | Когда использовать    |
| ------------------------------ | ----------- | ------------ | ----------------------- | -------------------------- | ------------------- | --------------------- |
| Operation / [[BlockOperation]] | Да          | Да           | Да (через queue)        | Нет (нужно вручную)        | Legacy              | Зависимости, миграция |
| [[actor]] + [[Task]]           | Частично    | Да           | Да (через логику)       | Да                         | Основной выбор      | Изменяемое состояние  |
| [[TaskGroup]]                  | Нет         | Да           | Да (addTask лимит)      | Да                         | Параллельные задачи | Асинхронные загрузки  |
| [[async let]]                  | Нет         | Да           | Да (параллельно)        | Да                         | Простые сценарии    | 2–5 независимых задач |

**Вывод 2026**:
- **Новый код** → **TaskGroup** / **actor** + **async let**  
- **Сложные зависимости / отмена** → **OperationQueue** (если не хотите переписывать)  
- Полная миграция → всё на [[Swift Concurrency]]

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — Operation уже считается legacy  
- **maxConcurrentOperationCount** — всегда ограничивайте (4–8 в большинстве случаев)  
- **isCancelled** — проверяйте в начале каждой операции и в тяжёлых циклах  
- **completionBlock** — используйте только для финального логирования / UI-обновлений  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `XCTestExpectation` + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных в операции

**Короткий девиз 2026**:
> «Operation — это когда нужны зависимости, отмена и контроль в старом стиле.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — TaskGroup + actor + async let + strict concurrency.  
> Если ты всё ещё используешь Operation в новом коде — спроси себя: «А точно ли это нужно?»»
