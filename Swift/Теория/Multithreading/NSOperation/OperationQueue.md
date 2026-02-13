### 1. Что такое OperationQueue и зачем она нужна

**OperationQueue** — это **высокоуровневый инструмент** для управления **очередью операций** (Operation / [[BlockOperation]]), который появился ещё в [[Objective-C]] и до сих пор активно используется в 2026 году.

Основные преимущества над [[GCD]] / [[DispatchQueue]]:

- **Зависимости** между операциями (можно сказать: «операция B должна начаться только после завершения A и C»)  
- **Отмена** операций (cancel) с возможностью проверки внутри операции  
- **Ограничение параллелизма** (`maxConcurrentOperationCount`)  
- **Удобная работа с результатами** ([[completionBlock]], [[KVO]], состояние finished/ready/executing)  
- **Поддержка приоритетов** внутри операций (queuePriority)  
- **Легко комбинировать** с GCD и Swift Concurrency

**Главные сценарии использования в 2026 году**:

- Сложные цепочки зависимостей (загрузка → обработка → сохранение → UI)  
- Параллельная обработка с ограничением (не больше 4 загрузок одновременно)  
- Отмена операций при уходе с экрана / смене состояния  
- Миграция старого кода на Swift Concurrency  
- Работа с legacy-библиотеками, которые используют Operation

### 2. Основные классы и свойства

| Класс / Свойство                    | Что это                                                        | Самый частый use-case в 2026        |
| ----------------------------------- | -------------------------------------------------------------- | ----------------------------------- |
| **OperationQueue**                  | Очередь операций                                               | Основной объект                     |
| **[[maxConcurrentOperationCount]]** | Максимум параллельных операций (по умолчанию  -1 = нет лимита) | Ограничение concurrency             |
| **qualityOfService**                | [[QoS]] всей очереди                                           | .utility, .userInitiated            |
| **[[Operation]]**                   | Базовый класс операции (абстрактный)                           | Наследование для кастомных операций |
| **[[BlockOperation]]**              | Простая операция из замыкания                                  | Быстрые задачи без зависимостей     |
| **[[isAsynchronous]]**              | true, если операция асинхронная                                | Для сетевых / I/O операций          |
| **[[completionBlock]]**             | Замыкание после завершения операции                            | Обновление UI / логирование         |
| **[[queuePriority]]**               | Приоритет внутри очереди (.veryLow … .veryHigh)                | Тонкая настройка внутри очереди     |

### 3. Самые популярные шаблоны использования в 2026 году

#### Шаблон 1 — Параллельная загрузка с лимитом (самый частый)

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4          // не больше 4 одновременно
queue.qualityOfService = .utility

var images: [UIImage?] = []

for url in imageURLs {
    let operation = BlockOperation {
        guard let data = try? Data(contentsOf: url),
              let image = UIImage(data: data) else { return }
        
        // безопасное обновление UI
        DispatchQueue.main.async {
            images.append(image)
            collectionView.reloadData()
        }
    }
    
    queue.addOperation(operation)
}

// Когда все закончены
queue.addBarrierBlock {
    DispatchQueue.main.async {
        print("Все \(images.count) изображений загружены")
    }
}
```

#### Шаблон 2 — Зависимости между операциями

```swift
let queue = OperationQueue()

let fetchOperation = BlockOperation {
    let data = fetchDataSync()
    // сохраняем в промежуточный результат
}

let processOperation = BlockOperation {
    let processed = processDataSync()
    // сохраняем результат
}

let saveOperation = BlockOperation {
    saveToDiskSync()
}

// Устанавливаем зависимости
processOperation.addDependency(fetchOperation)
saveOperation.addDependency(processOperation)

queue.addOperations([fetchOperation, processOperation, saveOperation], waitUntilFinished: false)

queue.addBarrierBlock {
    DispatchQueue.main.async {
        self.showSuccessMessage()
    }
}
```

#### Шаблон 3 — Отмена операций при уходе с экрана

```swift
class GalleryViewController: UIViewController {
    private let queue = OperationQueue()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        queue.maxConcurrentOperationCount = 3
        
        for url in urls {
            let op = BlockOperation { [weak self] in
                guard let self else { return }
                
                if op.isCancelled { return }
                
                let image = downloadImage(url)
                
                DispatchQueue.main.async {
                    if op.isCancelled { return }
                    self.collectionView.insertImage(image)
                }
            }
            
            queue.addOperation(op)
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        queue.cancelAllOperations()  // отменяем все
    }
}
```

### 4. OperationQueue vs современные альтернативы (2026 сравнение)

| Механизм                              | Зависимости | Отмена задач | Ограничение concurrency | Потокобезопасность | Рекомендация 2026   | Когда использовать            |
| ------------------------------------- | ----------- | ------------ | ----------------------- | ------------------ | ------------------- | ----------------------------- |
| OperationQueue                        | Да          | Да           | Да (maxConcurrent)      | Ручная             | Legacy              | Сложные зависимости, миграция |
| [[actor]] + [[Task]]                  | Частично    | Да           | Да (через логику)       | Встроенная         | Основной выбор      | Изменяемое состояние          |
| [[TaskGroup]]                         | Нет         | Да           | Да (addTask лимит)      | Встроенная         | Параллельные задачи | Асинхронные загрузки          |
| [[async let]]                         | Нет         | Да           | Да (параллельно)        | Встроенная         | Простые сценарии    | 2–5 независимых задач         |
| [[DispatchGroup]] + [[DispatchQueue]] | Нет         | Ограниченно  | Нет                     | Ручная             | Legacy              | Простые ожидания              |

**Вывод 2026**:
- **Новый код** → **TaskGroup** / **actor** + **async let**  
- **Сложные зависимости** → **OperationQueue** (если не хотите переписывать)  
- **Полная миграция** → всё на Swift Concurrency

### 5. Типичные ошибки с OperationQueue в 2026

| Ошибка                                            | Последствия                               | Как избежать                                       |
| ------------------------------------------------- | ----------------------------------------- | -------------------------------------------------- |
| Забыть вызвать [[completionBlock]]                | Операция «зависает» в очереди             | Использовать defer или явно вызывать               |
| Долгая работа без проверки [[isCancelled]]        | Операция продолжает работать после отмены | Проверять `if isCancelled { return }`              |
| Обновление UI напрямую из операции                | Main Thread Violation                     | Всегда `DispatchQueue.main.async` или `@MainActor` |
| [[maxConcurrentOperationCount]] = -1 (без лимита) | Thread Explosion                          | Устанавливать 4–8 в большинстве случаев            |
| Использование [[sync]] внутри операции            | Deadlock                                  | Только [[async]]                                   |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — OperationQueue уже считается legacy  
- **maxConcurrentOperationCount** — всегда ограничивайте (4–8 для большинства случаев)  
- **isCancelled** — проверяйте в начале каждой операции и в тяжёлых циклах  
- **completionBlock** — используйте для логирования / UI-обновлений  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `XCTestExpectation` + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных в операции

**Короткий девиз 2026**:
> «OperationQueue — это когда нужны зависимости, отмена и ограничение параллелизма в старом стиле.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — TaskGroup + actor + async let + strict concurrency.  
> Если ты всё ещё используешь OperationQueue в новом коде — спроси себя: «А точно ли это нужно?»»
