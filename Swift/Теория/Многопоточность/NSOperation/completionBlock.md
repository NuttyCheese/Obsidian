**completionBlock** — это **замыкание** ([[closure]]), которое автоматически вызывается **один раз** после завершения выполнения операции (`isFinished` становится `true`), независимо от того:

- операция завершилась **успешно**  
- была **отменена** (`cancel()`)  
- завершилась с **ошибкой**  
- была завершена **нормально**

**Ключевые характеристики в 2026 году**:

- Вызывается **один раз** в конце жизненного цикла операции  
- Выполняется **на той же очереди**, на которой выполнялась операция (если не переопределить)  
- **Не блокирует** очередь (выполняется асинхронно относительно основной работы операции)  
- **Не передаёт** результат / ошибку — для этого нужно использовать свои свойства или [[KVO]]  
- **Может быть изменён** даже после добавления операции в очередь (но лучше задавать до)  
- **Не вызывается**, если операция **никогда не запускалась** (например, отменена до старта)

### 2. Когда использовать completionBlock в 2026 году

| Сценарий                                       | Почему именно completionBlock                 | Альтернатива в 2026 (рекомендуемая) |
| ---------------------------------------------- | --------------------------------------------- | ----------------------------------- |
| Логирование завершения операции                | Гарантированно вызывается после finish        | [[Task]] + [[defer]] или `finally`  |
| Обновление UI после завершения задачи          | Простой способ сделать что-то после           | [[@MainActor]] + `Task` completion  |
| Очистка ресурсов после операции                | Надёжное место для [[deinit]]-подобной логики | `actor` + `deinit` или `Task`       |
| Запуск следующей операции (без зависимостей)   | Простой «then»-эффект                         | `async let` / `TaskGroup`           |
| Legacy-код / миграция на [[Swift Concurrency]] | Уже используется в старом коде                | Переписывать на `Task` / `actor`    |

**Важно**:  
в 2026 году `completionBlock` считается **legacy-подходом**.  
В новом коде почти всегда лучше использовать:
- `async` / `await` + `defer` / `finally`  
- `TaskGroup`  
- `actor` + `Task`

### 3. Самые популярные шаблоны использования в 2026

#### Шаблон 1 — Логирование + UI-обновление

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 4

let operation = BlockOperation {
    // Тяжёлая работа в фоне
    let data = fetchDataSync()
    
    // Безопасное обновление UI
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}

operation.completionBlock = {
    print("Операция завершена: \(Date())")
    
    // Дополнительное действие после завершения
    DispatchQueue.main.async {
        self.activityIndicator.stopAnimating()
    }
}

queue.addOperation(operation)
```

#### Шаблон 2 — Отмена и проверка в completionBlock

```swift
let operation = BlockOperation { [weak self] in
    guard let self else { return }
    
    for i in 1...1000 {
        if operation.isCancelled {
            print("Операция отменена на шаге \(i)")
            return
        }
        // тяжёлая работа
        Thread.sleep(forTimeInterval: 0.01)
    }
}

operation.completionBlock = { [weak self] in
    guard let self else { return }
    
    DispatchQueue.main.async {
        if operation.isCancelled {
            self.showMessage("Загрузка отменена")
        } else {
            self.showMessage("Загрузка завершена успешно")
        }
    }
}

queue.addOperation(operation)

// Отмена при уходе с экрана
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    operation.cancel()
}
```

#### Шаблон 3 — Несколько блоков + completionBlock

```swift
let operation = BlockOperation()

operation.addExecutionBlock {
    print("Шаг 1: загрузка")
}

operation.addExecutionBlock {
    print("Шаг 2: обработка")
}

operation.completionBlock = {
    print("Все шаги завершены")
    DispatchQueue.main.async {
        self.tableView.reloadData()
    }
}

queue.addOperation(operation)
```

### 4. Типичные ошибки с completionBlock в 2026

| Ошибка                                                 | Последствия                                                     | Как избежать                                               |
| ------------------------------------------------------ | --------------------------------------------------------------- | ---------------------------------------------------------- |
| Обновление UI напрямую в completionBlock               | [[Main Thread Violation]]                                       | Всегда `DispatchQueue.main.async` или `@MainActor`         |
| Сильная зависимость от completionBlock                 | Сложно тестировать, сложно отлаживать                           | Переходить на `async` / `await`                            |
| Забыть [weak self] в completionBlock                   | Утечка памяти                                                   | Всегда `[weak self]`                                       |
| Изменение completionBlock после добавления в очередь   | Неопределённое поведение                                        | Задавать до addOperation                                   |
| Использование completionBlock для асинхронных операций | Операция завершается до завершения вложенной асинхронной задачи | Использовать асинхронные операции или `isFinished` вручную |

### 5. BlockOperation + completionBlock vs современные альтернативы (2026 сравнение)

| Механизм                  | Зависимости | Отмена задач | Completion / finally | Потокобезопасность | Рекомендация 2026 | Когда использовать |
|---------------------------|-------------|--------------|-----------------------|---------------------|-------------------|---------------------|
| BlockOperation + completionBlock | Да          | Да           | Да (completionBlock)  | Ручная              | Legacy            | Зависимости, миграция |
| actor + Task              | Частично    | Да           | Да (defer / finally)  | Встроенная          | Основной выбор    | Изменяемое состояние |
| TaskGroup                 | Нет         | Да           | Да (await group.waitForAll()) | Встроенная          | Параллельные задачи | Асинхронные загрузки |
| async let                 | Нет         | Да           | Да (try await)        | Встроенная          | Простые сценарии   | 2–5 независимых задач |

**Вывод 2026**:
- **Новый код** → **TaskGroup** / **actor** + **async let** + `defer` / `finally`  
- **Сложные зависимости / отмена** → **OperationQueue + BlockOperation** (если не хотите переписывать)  
- Полная миграция → всё на Swift Concurrency

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — [[BlockOperation]] + completionBlock уже считается legacy  
- **[[maxConcurrentOperationCount]]** — всегда ограничивайте (4–8 в большинстве случаев)  
- **[[isCancelled]]** — проверяйте в начале каждой операции и в тяжёлых циклах  
- **completionBlock** — используйте только для финального логирования / UI-обновлений  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `XCTestExpectation` + `waitUntilFinished`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности операций в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасную передачу данных в операции

**Короткий девиз 2026**:
> «completionBlock — это когда нужно сделать что-то после завершения операции в старом стиле.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — defer / finally в Task + actor + TaskGroup.  
> Если ты всё ещё используешь completionBlock в новом коде — спроси себя: «А точно ли это нужно?»»
