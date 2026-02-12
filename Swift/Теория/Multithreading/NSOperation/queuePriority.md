Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по свойству **`queuePriority`** в классе `Operation` (и его подклассах, включая `BlockOperation`) в Swift.

### 1. Что такое queuePriority

**queuePriority** — это свойство типа `Operation.QueuePriority`, которое задаёт **относительный приоритет выполнения операции внутри одной `OperationQueue`**.

Оно влияет на то, **в каком порядке** операции будут запускаться, когда очередь имеет **несколько готовых к выполнению задач** и ограниченное количество параллельных слотов (`maxConcurrentOperationCount`).

**Ключевые факты 2026 года**:

- `queuePriority` работает **только внутри одной очереди**  
- Он **не влияет** на глобальный приоритет системы (QoS) — это разные вещи  
- Значение по умолчанию: `.normal`  
- Значение можно менять **до** добавления операции в очередь (после добавления изменение игнорируется)  
- `queuePriority` **не отменяет** зависимости (`dependencies`) — зависимые операции всё равно ждут  

### 2. Все возможные значения QueuePriority

| Значение                  | Числовое значение | Когда использовать (рекомендация 2026)                          | Относительный приоритет внутри очереди |
|---------------------------|-------------------|------------------------------------------------------------------|----------------------------------------|
| `.veryLow`                | -8                | Очень низкий приоритет (фоновые задачи, которые могут ждать)     | Самый низкий                           |
| `.low`                    | -4                | Низкоприоритетные задачи (аналитика, логирование)                | Низкий                                 |
| `.normal` (по умолчанию)  | 0                 | Обычные задачи без особой срочности                              | Средний                                |
| `.high`                   | +4                | Важные задачи, которые пользователь ждёт                         | Высокий                                |
| `.veryHigh`               | +8                | Критические задачи внутри очереди (например, UI-подготовка)      | Самый высокий                          |

**Важно**:
> `queuePriority` **не влияет** на глобальный QoS (`.userInitiated`, `.background` и т.д.).  
> Это **локальный** приоритет **только внутри одной очереди**.

### 3. Как правильно использовать queuePriority

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 3
queue.qualityOfService = .utility

let op1 = BlockOperation { print("Операция 1") }
op1.queuePriority = .veryHigh

let op2 = BlockOperation { print("Операция 2") }
op2.queuePriority = .normal

let op3 = BlockOperation { print("Операция 3") }
op3.queuePriority = .veryLow

queue.addOperations([op1, op2, op3], waitUntilFinished: false)
```

**Возможный порядок запуска** (примерный):
- Сначала op1 (veryHigh)  
- Затем op2 (normal)  
- Затем op3 (veryLow)

**Но**:
- Порядок **не гарантирован на 100%** — зависит от состояния очереди и планировщика  
- Если `maxConcurrentOperationCount = 1`, приоритет вообще не влияет (всё равно по порядку добавления)

### 4. Самые популярные шаблоны использования в 2026 году

#### Шаблон 1 — Приоритет внутри очереди загрузки изображений

```swift
let imageQueue = OperationQueue()
imageQueue.maxConcurrentOperationCount = 4
imageQueue.qualityOfService = .utility

// Критическое изображение (обложка)
let coverOp = BlockOperation { /* загрузка обложки */ }
coverOp.queuePriority = .veryHigh

// Обычные изображения
let imageOps = imageURLs.map { url in
    let op = BlockOperation { /* загрузка изображения */ }
    op.queuePriority = .normal
    return op
}

// Фоновые миниатюры
let thumbnailOps = thumbnailURLs.map { url in
    let op = BlockOperation { /* генерация миниатюры */ }
    op.queuePriority = .low
    return op
}

imageQueue.addOperations([coverOp] + imageOps + thumbnailOps, waitUntilFinished: false)
```

#### Шаблон 2 — Приоритет + зависимость

```swift
let fetchOp = BlockOperation { /* загрузка данных */ }
fetchOp.queuePriority = .veryHigh

let processOp = BlockOperation { /* обработка */ }
processOp.queuePriority = .high
processOp.addDependency(fetchOp)

let saveOp = BlockOperation { /* сохранение */ }
saveOp.queuePriority = .normal
saveOp.addDependency(processOp)

queue.addOperations([fetchOp, processOp, saveOp], waitUntilFinished: false)
```

### 5. Типичные ошибки с queuePriority в 2026

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Менять queuePriority после добавления в очередь | Изменение игнорируется                   | Устанавливать до `addOperation` |
| Думать, что queuePriority влияет на глобальный QoS | Нет эффекта на энергопотребление / вытеснение | Помнить: queuePriority — только внутри очереди |
| Использовать очень высокие приоритеты везде | Все операции становятся «равными» → приоритет теряет смысл | Использовать умеренно (.veryHigh только для 1–2 критических задач) |
| Забыть про maxConcurrentOperationCount      | Приоритет не имеет эффекта при -1        | Всегда явно задавать лимит (4–8) |
| Ожидать строгого порядка по queuePriority  | Порядок не гарантирован на 100%          | Для строгого порядка — использовать зависимости или сериальную очередь |

### 6. queuePriority vs современные альтернативы (2026 сравнение)

| Механизм                  | Локальный приоритет внутри очереди | Глобальный приоритет (QoS) | Отмена задач | Рекомендация 2026 | Когда использовать |
|---------------------------|-------------------------------------|-----------------------------|--------------|-------------------|---------------------|
| Operation.queuePriority   | Да                                  | Да (наследуется от queue)   | Да           | Legacy            | Зависимости, миграция |
| Task.priority             | Нет (только глобальный)             | Да (Task priority)          | Да           | Основной выбор    | Асинхронные задачи |
| actor                     | Нет (1 поток)                       | Да (через Task)             | Да           | Основной выбор    | Изменяемое состояние |
| DispatchQueue QoS         | Нет                                 | Да                          | Ограниченно  | Legacy            | Простые фоновые задачи |

**Вывод 2026**:
- **queuePriority** — полезен только в legacy-коде с `OperationQueue`  
- В новом коде приоритет задаётся через `Task(priority:)`  
- Для строгого порядка → используйте `dependencies` или `actor`

### 7. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — OperationQueue + queuePriority уже legacy  
- **queuePriority** — используйте умеренно (`.veryHigh` только для 1–2 критических операций)  
- **maxConcurrentOperationCount** — всегда ограничивайте (4–8 в большинстве случаев)  
- **isCancelled** — проверяйте в начале каждой операции  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `XCTestExpectation` + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных в операции

**Короткий девиз 2026**:
> «queuePriority — это когда нужно сказать внутри очереди: «эта операция важнее других».  
> В 2026 году это уже legacy-механизм.  
> Новый стандарт — Task(priority:) + actor + TaskGroup.  
> Если ты всё ещё используешь queuePriority в новом коде — спроси себя: «А точно ли это нужно?»»

Удачи с управляемыми приоритетами и отзывчивым кодом в Swift! 🚀