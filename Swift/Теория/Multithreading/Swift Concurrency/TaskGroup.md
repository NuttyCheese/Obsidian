### 1. Что такое TaskGroup и зачем он нужен

**`TaskGroup`** — это **структурированный контейнер** для **параллельного выполнения нескольких асинхронных задач** (child tasks).

Он позволяет:

- динамически добавлять задачи (`addTask`)  
- собирать их результаты по мере готовности (`for await ... in group`)  
- **автоматически** ждать завершения всех дочерних задач при выходе из блока `withTaskGroup`  
- **автоматически** отменять все дочерние задачи, если родительская задача отменяется  
- обрабатывать ошибки через `withThrowingTaskGroup`

**Главные преимущества TaskGroup** (2026):

- **структурированная конкурентность** — все дочерние задачи завершаются вместе с родительской  
- **динамическое количество задач** — можно добавлять задачи в цикле, по условию, во время выполнения  
- **эффективное ожидание** — не нужно вручную хранить массив задач и ждать каждую  
- **отмена по цепочке** — `parentTask.cancel()` отменяет всю группу  
- **низкие накладные расходы** — задачи — это лёгкие подзадачи, а не полноценные `Task`

**Коротко**:
> TaskGroup = «я запускаю много асинхронных задач параллельно, собираю результаты по мере готовности и гарантирую, что все они завершатся (или отменятся) вместе со мной».

### 2. Основные виды TaskGroup (2026)

| Вид                                | Возвращаемый тип элементов | Поддержка ошибок | Когда использовать                     | Синтаксис |
|------------------------------------|-----------------------------|-------------------|----------------------------------------|-----------|
| `withTaskGroup(of: T.self)`        | `T`                         | Нет               | Задачи без ошибок                      | `await withTaskGroup` |
| `withThrowingTaskGroup(of: T.self)`| `T`                         | Да                | Задачи, которые могут бросать `Error`  | `try await withThrowingTaskGroup` |
| `withTaskGroup(of: Void.self)`     | `Void`                      | Нет               | Просто параллельный запуск без результатов | `await withTaskGroup` |

### 3. Основной синтаксис и методы

```swift
await withTaskGroup(of: Int.self) { group in
    // Добавление задач
    group.addTask { 1 }
    group.addTask { await someAsync() }
    
    // Перебор результатов по мере готовности
    for await result in group {
        print(result)
    }
    
    // Автоматическое ожидание всех задач при выходе из блока
}
```

Ключевые методы `TaskGroup`:

| Метод                              | Что делает                                     | Возвращает          | Примечание                       |
| ---------------------------------- | ---------------------------------------------- | ------------------- | -------------------------------- |
| `addTask { ... }`                  | Добавляет новую дочернюю задачу                | `Task<Void, Never>` | Можно вызывать в цикле           |
| `addTask(priority: .high) { ... }` | С явным приоритетом                            | `Task<Void, Never>` | `.high`, `.userInitiated` и т.д. |
| `for await result in group`        | Перебирает результаты по мере завершения задач | `Element` (T)       | Порядок не гарантирован          |
| `group.next()`                     | Получает следующий результат (асинхронно)      | `Element?`          | Ручной вариант [[for-await]]     |
| `group.cancelAll()`                | Отменяет все дочерние задачи                   | Void                | Полезно для ранней отмены        |
| `group.isCancelled`                | Проверяет, отменена ли группа                  | Bool                | Полезно внутри задач             |

### 4. Самые популярные шаблоны TaskGroup 2026 года

#### Шаблон 1 — Параллельная загрузка + сбор результатов

```swift
func fetchUser(id: Int) async throws -> User { ... }
func fetchPosts(userId: Int) async throws -> [Post] { ... }

@MainActor
class FeedViewModel: ObservableObject {
    @Published var user: User?
    @Published var posts: [Post] = []

    func load(for userId: Int) async throws {
        try await withThrowingTaskGroup(of: Void.self) { group in
            group.addTask {
                let user = try await fetchUser(id: userId)
                await MainActor.run { self.user = user }
            }
            
            group.addTask {
                let posts = try await fetchPosts(userId: userId)
                await MainActor.run { self.posts = posts }
            }
            
            // Ждём все задачи
            try await group.waitForAll()
        }
    }
}
```

#### Шаблон 2 — Динамическое добавление задач + сбор результатов

```swift
func processImage(url: URL) async -> UIImage? { ... }

Task {
    let urls = ["url1", "url2", "url3", "url4"]
    
    let images = await withTaskGroup(of: UIImage?.self) { group in
        for url in urls {
            group.addTask {
                await processImage(url: URL(string: url)!)
            }
        }
        
        var results: [UIImage?] = []
        for await image in group {
            results.append(image)
        }
        return results
    }
    
    await MainActor.run {
        collectionView.insertImages(images.compactMap { $0 })
    }
}
```

#### Шаблон 3 — TaskGroup с ранней отменой и обработкой ошибок

```swift
func fetchWithTimeout<T>(timeout: Duration, operation: @escaping () async throws -> T) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask {
            try await operation()
        }
        
        group.addTask {
            try await Task.sleep(for: timeout)
            throw CancellationError()
        }
        
        for try await result in group {
            group.cancelAll()  // отменяем все остальные задачи
            return result
        }
        
        throw CancellationError()  // если таймаут сработал первым
    }
}
```

### 5. Типичные ошибки и ловушки 2026 года

| Ошибка                                     | Последствия                         | Как избежать                                |
| ------------------------------------------ | ----------------------------------- | ------------------------------------------- |
| Забыть `await` перед `withTaskGroup`       | Ошибка компиляции                   | Всегда `await withTaskGroup`                |
| Добавить задачу после `for await`          | Задача не выполнится                | Добавляй задачи **до** перебора             |
| Не дождаться всех результатов              | Предупреждение компилятора / утечка | Всегда `for await` или `group.waitForAll()` |
| Делать тяжёлую синхронную работу в addTask | Блокирует [[executor]]              | Тяжёлое — в отдельный `Task` внутри         |
| Забыть отменять группу при ошибке          | Лишние задачи продолжают работать   | `group.cancelAll()` при ошибке              |
| Использовать TaskGroup для 2–3 задач       | Избыточно                           | Для 2–6 задач лучше `async let`             |

### 6. TaskGroup vs другие механизмы параллелизма (2026 сравнение)

| Механизм                          | Количество задач  | Динамическое добавление | Отмена наследуется | Обработка ошибок | Рекомендация 2026           | Когда использовать       |
| --------------------------------- | ----------------- | ----------------------- | ------------------ | ---------------- | --------------------------- | ------------------------ |
| [[async let]]                     | 2–6 (фиксировано) | Нет                     | Да                 | [[try]] await    | Простые параллельные вызовы | 2–6 независимых задач    |
| `withTaskGroup`                   | Любое             | Да                      | Да                 | try await        | Основной выбор              | Динамическое число задач |
| `TaskGroup.addTask`               | Любое             | Да                      | Да                 | try await        | Масштабируемость            | Много задач              |
| Последовательные `await`          | Любое             | —                       | Да                 | try await        | Зависимые задачи            | Цепочки операций         |
| `DispatchQueue.concurrentPerform` | Любое             | Нет                     | Нет                | Ручная           | Legacy                      | Старый код               |

**Вывод 2026**:
- `async let` — для **фиксированного** небольшого числа параллельных задач  
- `withTaskGroup` — для **динамического** числа задач, сложной логики и отмены  
- `TaskGroup` — **основной** инструмент для масштабируемого параллелизма в Swift 6+

### 7. Лучшие практики 2026 года

- **Используй `async let`** для 2–6 независимых задач — проще и читаемее  
- **Используй `withTaskGroup`** когда:
  - количество задач неизвестно заранее  
  - нужно динамически добавлять задачи  
  - нужна ранняя отмена или обработка ошибок  
- **Всегда** дожидайся результатов (`for await` или `group.waitForAll()`)  
- **Обрабатывай отмену** — проверяй `Task.isCancelled` внутри задач  
- **UI** — финальное присваивание делай на `@MainActor`  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые `await`  
- **Мониторинг** — Instruments → Swift Tasks — смотри количество задач и время выполнения

**Короткий девиз 2026**:
> «TaskGroup — это когда ты говоришь: «запускай сколько угодно задач параллельно, собирай результаты по мере готовности, и отменяй всех, если я отменюсь».  
> В 2026 году это **основной** инструмент для сложного параллельного асинхронного кода в Swift.»
