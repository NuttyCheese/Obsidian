GCD остаётся **одним из самых важных** и **самых часто используемых** механизмов многопоточности в [[iOS]]/macOS-приложениях, но его роль сильно изменилась.

| Период    | Роль GCD                                        | Рекомендация Apple в 2026 году                                                  |
| --------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| 2009–2020 | Основной инструмент многопоточности             | —                                                                               |
| 2021–2024 | Основной → параллельно со [[Swift Concurrency]] | Переходный период                                                               |
| 2025–2026 | Legacy + очень частый инструмент                | **Swift Concurrency — основной**, GCD — для совместимости и специфических задач |

**Ключевой вывод 2026**:
> В новом коде GCD используют **только там**, где Swift Concurrency пока не может полностью заменить (или неудобно):
> - работа с очень старыми [[callback]]-[[API]]  
> - тонкая настройка [QoS] и приоритетов  
> - барьерные операции в [[concurrent]]-очередях  
> - интеграция с низкоуровневыми C-библиотеками  
> - максимальная производительность в горячих участках

### 2. Основные сущности GCD в 2026 году (таблица)

| Сущность                                  | Что это                                 | Актуальность в 2026 | Лучшая альтернатива в [[Swift Concurrency]] |
| ----------------------------------------- | --------------------------------------- | ------------------- | ------------------------------------------- |
| [[DispatchQueue]].[[main]]                | Главная очередь (UI)                    | ★★★★★               | `@MainActor` / `MainActor.run`              |
| [[DispatchQueue]].[[global]]([[QoS]]:)    | Глобальные очереди по приоритету        | ★★★★☆               | `Task(priority:)`                           |
| `DispatchQueue(label:)`                   | Пользовательская очередь                | ★★★★☆               | [[actor]] или сериальный `DispatchQueue`    |
| [[async]] / [[sync]]                      | Асинхронный / синхронный [[dispatch]]   | ★★★★☆               | [[await]] / почти никогда `sync`            |
| [[asyncAfter]]                            | Отложенный dispatch                     | ★★★☆☆               | `Task.sleep()`                              |
| [[DispatchGroup]]                         | Ожидание группы задач                   | ★★★☆☆               | `withTaskGroup` / [[async let]]             |
| [[DispatchSemaphore]]                     | Семафор                                 | ★★☆☆☆               | `actor` + `withTaskGroup`                   |
| [[DispatchWorkItem]]                      | Отменяемая задача                       | ★★☆☆☆               | `Task` + `cancel()`                         |
| [[DispatchBarrier\|Barrier]] (`.barrier`) | Барьерная операция в concurrent-очереди | ★★★★☆               | `actor` (самый частый переход)              |

### 3. Самые актуальные и рекомендуемые паттерны GCD в 2026 году

#### Паттерн 1 — Фон → UI (самый частый шаблон)

```swift
// 2026 — всё ещё очень часто встречается
func loadUser() {
    DispatchQueue.global(qos: .userInitiated).async {
        let user = fetchUserFromNetwork()
        
        DispatchQueue.main.async {
            self.nameLabel.text = user.name
            self.tableView.reloadData()
        }
    }
}
```

**Современный эквивалент** (рекомендуется):

```swift
@MainActor
class UserViewModel: ObservableObject {
    @Published var name = ""
    
    func load() async {
        let user = await fetchUser()
        name = user.name  // безопасно — @MainActor
    }
}
```

#### Паттерн 2 — Барьерная запись (последний оплот GCD)

```swift
// 2026 — один из немногих случаев, где GCD всё ещё часто предпочтительнее
let queue = DispatchQueue(label: "cache", attributes: .concurrent)
var cache: [String: Data] = [:]

func saveToCache(_ data: Data, key: String) {
    queue.async(flags: .barrier) {
        cache[key] = data
    }
}

func getFromCache(_ key: String) -> Data? {
    queue.sync { cache[key] }
}
```

**Современный эквивалент** (почти всегда лучше):

```swift
actor Cache {
    private var storage: [String: Data] = [:]
    
    func save(_ data: Data, key: String) {
        storage[key] = data
    }
    
    func get(_ key: String) -> Data? {
        storage[key]
    }
}
```

#### Паттерн 3 — [[DispatchGroup]] → замена на [[TaskGroup]]

```swift
// Старый GCD-стиль (всё ещё встречается)
let group = DispatchGroup()

group.enter()
DispatchQueue.global().async {
    // задача 1
    group.leave()
}

group.enter()
DispatchQueue.global().async {
    // задача 2
    group.leave()
}

group.notify(queue: .main) {
    print("Все задачи завершены")
}
```

**Современный стиль** (рекомендуется):

```swift
await withTaskGroup(of: Void.self) { group in
    group.addTask {
        await task1()
    }
    
    group.addTask {
        await task2()
    }
    
    await group.waitForAll()
    
    await MainActor.run {
        print("Все задачи завершены")
    }
}
```

### 4. Когда GCD всё ещё побеждает в 2026 году

| Сценарий                                         | Почему GCD пока выигрывает         | Альтернатива Swift Concurrency |
| ------------------------------------------------ | ---------------------------------- | ------------------------------ |
| Барьерная запись в concurrent-очереди            | Очень высокая производительность   | `actor` (обычно достаточно)    |
| Тонкая настройка [[QoS]] и приоритетов           | Максимальный контроль              | `Task(priority:)`              |
| Интеграция с очень старыми C-API                 | Прямой доступ к dispatch_* функции | `withCheckedContinuation`      |
| Максимальная производительность в горячих петлях | Минимальные накладные расходы      | `Atomic` + `actor`             |
| Поддержка iOS 12–14 (редко)                      | Совместимость                      | —                              |

### 5. Лучшие практики GCD в 2026 году

- **Переходите на Swift Concurrency** — GCD уже legacy  
- **Никогда** не используйте `sync` на `.main` из главного потока — **[[deadlock]]**  
- **Никогда** не используйте `sync` внутри той же очереди — **deadlock**  
- **QoS** — всегда указывайте явно (`.userInitiated`, `.utility`, `.background`)  
- **Барьер** — используйте только если actor не подходит (очень редкие случаи)  
- **Для UI** — только `@MainActor` / `MainActor.run`  
- **Для тестов** — используйте `await` и [[XCTestExpectation]]  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасное использование GCD

**Короткий девиз 2026**:
> «GCD — это когда тебе нужен максимальный контроль и минимальные накладные расходы.  
> В 2026 году это уже **legacy-инструмент**.  
> Новый стандарт — `Task`, `actor`, `@MainActor`, `TaskGroup`, `async let`.  
> Используй GCD только там, где Concurrency пока не закрывает задачу на 100% или для поддержки очень старого кода.»
