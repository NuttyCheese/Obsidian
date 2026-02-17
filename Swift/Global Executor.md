**Global Executor** — это **исполнитель задач по умолчанию** в [[Swift Concurrency]], который используется, когда вы **не указали** конкретный контекст выполнения (например `@MainActor`, `actor`, `Task.detached` или кастомный executor).

Он отвечает за выполнение большинства «обычных» асинхронных задач, которые запускаются через:

- `Task { ... }` (без `@MainActor` и без `detached`)  
- `async let`  
- обычные `async` функции без явной изоляции

**Ключевые характеристики Global Executor** (2026):

- Работает на **глобальном пуле потоков** (global thread pool)  
- Потоки берутся из системного пула [[GCD]] (libdispatch)  
- **Не привязан** к конкретному потоку — задача может выполняться на любом доступном потоке пула  
- **Не имеет изоляции** — нет гарантии, что две задачи не будут выполняться одновременно  
- **Максимально параллельный** — система старается запускать как можно больше задач одновременно  
- **Экономит ресурсы** — потоки переиспользуются, создаются/уничтожаются автоматически

**Коротко**:
> Global Executor = «любой свободный фоновый поток из системного пула, который сейчас не занят».

### 2. Когда используется Global Executor

| Ситуация                                       | Executor по умолчанию                   | Примечание                                               |
| ---------------------------------------------- | --------------------------------------- | -------------------------------------------------------- |
| `Task { ... }` без `@MainActor` или `detached` | Global Executor                         | Самый частый случай                                      |
| `async let a = fetchData()`                    | Global Executor                         | Все async let задачи идут параллельно на Global Executor |
| Обычная `async func` без аннотации             | Global Executor                         | Если вызвана из не-изолированного контекста              |
| `TaskGroup.addTask { ... }`                    | Global Executor                         | Задачи внутри [[TaskGroup]] по умолчанию на Global       |
| `withTaskGroup` / `withThrowingTaskGroup`      | Global Executor                         | То же самое                                              |
| `actor` метод, вызванный из не-actor контекста | Сначала Global → потом executor actor-а | Переключается при await                                  |

**Когда НЕ используется Global Executor**:

- `@MainActor` — всегда главный поток  
- внутри `actor` — собственный executor актёра  
- `Task.detached { ... }` — detached executor (особый подвид global, без текущего контекста)  
- `Task { @CustomActor in ... }` — executor кастомного глобального актёра

### 3. Самые популярные шаблоны 2026 года

#### Шаблон 1 — Обычная фоновая задача (самый частый)

```swift
Task {
    // Выполняется на Global Executor
    let data = heavyComputation()
    print("Данные:", data)
    
    await MainActor.run {
        // Переключаемся на MainActor
        label.text = data.description
    }
}
```

#### Шаблон 2 — Параллельные независимые задачи ([[async let]])

```swift
func fetchUser()    async -> User    { ... } // ~1.2 сек
func fetchPosts()   async -> [Post]  { ... } // ~0.9 сек
func fetchComments() async -> [Comment] { ... } // ~1.5 сек

Task {
    async let user     = fetchUser()
    async let posts    = fetchPosts()
    async let comments = fetchComments()

    // Все три задачи идут параллельно на Global Executor

    let combined = await (user: user, posts: posts, comments: comments)
    
    await MainActor.run {
        self.user = combined.user
        self.posts = combined.posts
        self.comments = combined.comments
    }
}
```

#### Шаблон 3 — TaskGroup на Global Executor

```swift
Task {
    await withTaskGroup(of: String.self) { group in
        for url in urls {
            group.addTask {
                await downloadImage(from: url) // на Global Executor
            }
        }
        
        var results: [String] = []
        for await path in group {
            results.append(path)
        }
        
        await MainActor.run {
            self.images = results
        }
    }
}
```

#### Шаблон 4 — Сравнение executor-ов

```swift
print("До:", Thread.current)

Task {
    print("Обычный Task → Global Executor:", Thread.current)
    
    await MainActor.run {
        print("MainActor.run → главный поток:", Thread.current)
    }
    
    await someActor.doWork()  // → executor someActor
}

Task.detached {
    print("Task.detached → detached executor:", Thread.current)
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                       | Последствия                            | Как избежать                                     |
| -------------------------------------------- | -------------------------------------- | ------------------------------------------------ |
| Доступ к actor-свойству без `await`          | Ошибка компиляции                      | Всегда `await actor.property`                    |
| Долгая синхронная работа на Global Executor  | Может блокировать один из потоков пула | Тяжёлое — в `actor` или `TaskGroup`              |
| Думать, что Global Executor — это один поток | Нет — это пул потоков                  | Он может использовать много потоков одновременно |
| Забыть переключиться на `@MainActor` для UI  | [[Main Thread Violation]]              | Все UI — через `@MainActor` / `MainActor.run`    |
| Слишком много `Task { ... }` без ограничения | Thread starvation / перегрузка         | Используй [[TaskGroup]] с лимитом или [[actor]]  |
| Захват non-Sendable типов в замыкании        | Ошибка strict concurrency              | Всё захватываемое должно быть [[Sendable]]       |

### 5. Global Executor vs другие executor-ы (2026 сравнение)

| Executor               | Поток(и) выполнения                 | Изоляция | Параллелизм    | Рекомендация 2026        | Когда использовать                               |
| ---------------------- | ----------------------------------- | -------- | -------------- | ------------------------ | ------------------------------------------------ |
| **Global Executor**    | Глобальный пул потоков [[GCD]]      | Нет      | Максимальный   | Основной для фона        | Фоновые вычисления, [[async let]], [[TaskGroup]] |
| **MainActor executor** | Только главный поток                | Полная   | Нет            | Для всего UI             | ViewModel, UI-обновления                         |
| **Actor executor**     | Любой поток из пула (свой на actor) | Полная   | Нет (1 за раз) | Для состояния            | Данные, сервисы, кэш                             |
| **Detached executor**  | Глобальный пул (без наследования)   | Нет      | Максимальный   | Максимальный параллелизм | Тяжёлые вычисления                               |
| **Custom executor**    | Любой (можно указать свой)          | Полная   | Зависит        | Редко                    | Низкоуровневый код                               |

### 6. Лучшие практики 2026 года

- **Global Executor** — используй по умолчанию для **всех фоновых независимых задач**  
- **UI** — всегда переключайся на `@MainActor` / `MainActor.run`  
- **Состояние** — храни в `actor` (не на Global Executor)  
- **Параллелизм** — `async let` (2–6 задач) или `TaskGroup` (много задач)  
- **Отмена** — используй `Task.cancel()` — Global Executor сам обработает  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит unsafe захваты  
- **Мониторинг** — Instruments → Swift Tasks — смотри, на каких executor-ах выполняются задачи

**Короткий девиз 2026**:
> «Global Executor — это когда ты говоришь: «сделай это в фоне, на любом свободном потоке, мне не важно на каком».  
> В 2026 году это основной «рабочий» executor для всего, что не UI и не требует строгой изоляции.»
