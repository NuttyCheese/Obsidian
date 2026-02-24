**BlockOperation** — это **конкретный** (concrete) подкласс класса [[Operation]], который позволяет создать операцию **из одного или нескольких замыканий** ([[closure]] / блоков кода).

Это самый простой и самый часто используемый способ создания операций в **[[OperationQueue]]**, когда вам не нужно сложное состояние, [[KVO]] или кастомная логика.

**Главные особенности BlockOperation в 2026 году**:

- создаётся из **одного или нескольких замыканий**  
- автоматически завершается, когда **все** замыкания выполнены  
- поддерживает **отмену** ([[isCancelled]])  
- можно добавить **зависимости** ([[addDependency]])  
- можно задать **приоритет** внутри очереди ([[queuePriority]])  
- можно указать **[[QoS]]** (qualityOfService)  
- **не асинхронный по умолчанию** ([[isAsynchronous]] = false) — если нужна асинхронность, нужно вручную управлять состоянием

### 2. Основные способы создания BlockOperation

| Способ создания                               | Когда использовать                              | Пример |
|-----------------------------------------------|--------------------------------------------------|--------|
| `init(block:)`                                | Одна простая задача                              | `BlockOperation { ... }` |
| `init()` + `addExecutionBlock(_:)`            | Несколько блоков (выполняются последовательно)   | Чаще всего |
| `BlockOperation { ... }` + `queuePriority`    | Задача с приоритетом внутри очереди              | Редко  |
| `BlockOperation { ... }` + `qualityOfService` | Явный QoS для всей операции                      | Рекомендуется |

### 3. Самые популярные шаблоны использования в 2026 году

#### Шаблон 1 — Простая фоновая операция с UI-обновлением

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4
queue.qualityOfService = .utility

let operation = BlockOperation {
    // Тяжёлая работа в фоне
    let data = fetchDataSync()
    
    // Безопасное обновление UI
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}

operation.completionBlock = {
    print("Операция завершена")
}

queue.addOperation(operation)
```

#### Шаблон 2 — Отмена операции при уходе с экрана

```swift
class ImageGalleryViewController: UIViewController {
    private let queue = OperationQueue()
    private var operations: [BlockOperation] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        queue.maxConcurrentOperationCount = 3
        
        for url in imageURLs {
            let op = BlockOperation { [weak self] in
                guard let self else { return }
                
                if op.isCancelled { return }
                
                guard let data = try? Data(contentsOf: url),
                      let image = UIImage(data: data) else { return }
                
                DispatchQueue.main.async {
                    if op.isCancelled { return }
                    self.collectionView.insertImage(image)
                }
            }
            
            operations.append(op)
            queue.addOperation(op)
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        queue.cancelAllOperations()          // отменяем все
        operations.forEach { $0.cancel() }   // дополнительно
    }
}
```

#### Шаблон 3 — Несколько блоков в одной операции (последовательное выполнение)

```swift
let operation = BlockOperation()

operation.addExecutionBlock {
    print("Шаг 1: загрузка данных")
    // ...
}

operation.addExecutionBlock {
    print("Шаг 2: обработка")
    // ...
}

operation.addExecutionBlock {
    print("Шаг 3: сохранение")
    // ...
}

operation.completionBlock = {
    DispatchQueue.main.async {
        print("Все шаги завершены")
    }
}

queue.addOperation(operation)
```

**Важно**: блоки в `addExecutionBlock` выполняются **последовательно** в той же операции.

### 4. BlockOperation vs другие механизмы (сравнение 2026)

| Механизм                  | Зависимости | Отмена задач | Ограничение concurrency | Асинхронность по умолчанию | Рекомендация 2026   | Когда использовать    |
| ------------------------- | ----------- | ------------ | ----------------------- | -------------------------- | ------------------- | --------------------- |
| BlockOperation            | Да          | Да           | Да (через queue)        | Нет (нужно вручную)        | Legacy              | Зависимости, миграция |
| [[Operation]] (кастомный) | Да          | Да           | Да (через queue)        | Да (если переопределить)   | Legacy              | Сложная логика        |
| [[actor]] + [[Task]]      | Частично    | Да           | Да (через логику)       | Да                         | Основной выбор      | Изменяемое состояние  |
| [[TaskGroup]]             | Нет         | Да           | Да (addTask лимит)      | Да                         | Параллельные задачи | Асинхронные загрузки  |
| [[async let]]             | Нет         | Да           | Да (параллельно)        | Да                         | Простые сценарии    | 2–5 независимых задач |

**Вывод 2026**:
- **Новый код** → **[[TaskGroup]]** / **[[actor]]** + **[[async let]]**  
- **Сложные зависимости / отмена** → **[[OperationQueue]] + BlockOperation** (если не хотите переписывать)  
- Полная миграция → всё на Swift Concurrency

### 5. Типичные ошибки с BlockOperation в 2026

| Ошибка                                            | Последствия                             | Как избежать                                       |
| ------------------------------------------------- | --------------------------------------- | -------------------------------------------------- |
| Забыть проверять `isCancelled`                    | Операция продолжает работу после отмены | Проверять в начале и в циклах                      |
| Долгая работа без проверки отмены                 | Задержка после отмены                   | `if isCancelled { return }`                        |
| Обновление UI напрямую из операции                | [[Main Thread Violation]]               | Всегда `DispatchQueue.main.async` или `@MainActor` |
| [[maxConcurrentOperationCount]] = -1 (без лимита) | [[Thread Explosion]]                    | Устанавливать 4–8                                  |
| Использование [[sync]] внутри операции            | [[deadlock]]                            | Только async                                       |
| Забыть вызвать [[completionBlock]]                | Операция «зависает» в очереди           | Использовать defer или явно вызывать               |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — BlockOperation уже считается legacy  
- **maxConcurrentOperationCount** — всегда ограничивайте (4–8 в большинстве случаев)  
- **isCancelled** — проверяйте в начале каждой операции и в тяжёлых циклах  
- **completionBlock** — используйте для логирования / UI-обновлений  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте [[XCTestExpectation]] + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных в операции

**Короткий девиз 2026**:
> «BlockOperation — это когда нужна простая операция с отменой и зависимостями в старом стиле.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — TaskGroup + actor + async let + strict concurrency.  
> Если ты всё ещё используешь BlockOperation в новом коде — спроси себя: «А точно ли это нужно?»»
