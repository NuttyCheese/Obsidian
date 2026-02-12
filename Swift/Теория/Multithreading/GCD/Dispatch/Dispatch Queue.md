Вот **полное, подробное и максимально насыщенное** руководство по **Dispatch Queues** в Swift и Grand Central Dispatch (GCD) — актуально на 2026 год.

### 1. Что такое Dispatch Queue (очередь диспетчера)

**Dispatch Queue** — это **очередь задач**, в которую вы помещаете замыкания (closures), а система GCD сама решает:

- **когда** их выполнять  
- **на каком потоке**  
- **параллельно или последовательно**  
- **с каким приоритетом**

Это **основной механизм** многопоточности в iOS/macOS/tvOS/watchOS с 2009 года и до сих пор (2026).

### 2. Типы очередей (2026 актуально)

| Тип очереди                        | Параллелизм          | Порядок выполнения | Поток выполнения               | Лучший use-case в 2026 году                     | Рекомендация Apple |
|------------------------------------|----------------------|---------------------|--------------------------------|--------------------------------------------------|----------------------|
| **Сериальная** (serial)            | 1 задача за раз      | Строго FIFO         | Один поток (может меняться)    | Защита mutable состояния, последовательность     | Да                   |
| **Конкурентная** (concurrent)      | Пул потоков (4–64+)  | Не гарантирован     | Пул потоков                    | Редко — почти всегда actor / TaskGroup лучше     | Почти нет            |
| **Глобальная** (global)            | Конкурентная         | Не гарантирован     | Системный пул потоков          | Фоновые независимые задачи                       | Да (основной выбор)  |
| **Главная** (main)                 | Сериальная           | Строго FIFO         | Главный поток (UI thread)      | Всё, что связано с UI                            | Да                   |

**Ключевой факт 2026**:
> Apple **очень сильно** продвигает **Swift Concurrency** (`Task`, `actor`, `TaskGroup`, `@MainActor`) как замену GCD.  
> DispatchQueue остаётся актуальным только для:
> - поддержки legacy-кода  
> - интеграции с очень старыми библиотеками  
> - очень простых фоновых операций

### 3. Создание очередей (правильные способы 2026)

```swift
// 1. Сериальная очередь (самый частый и рекомендуемый)
let serialQueue = DispatchQueue(label: "com.example.serial")

// 2. Сериальная с QoS (очень рекомендуется)
let serialQueue = DispatchQueue(
    label: "com.example.serial",
    qos: .utility
)

// 3. Глобальная очередь (самый частый для фона)
let backgroundQueue = DispatchQueue.global(qos: .background)

// 4. Главная очередь (только для UI)
let mainQueue = DispatchQueue.main

// 5. Конкурентная очередь (почти никогда не используйте в 2026)
let concurrentQueue = DispatchQueue(
    label: "com.example.concurrent",
    attributes: .concurrent
)
```

**Правило 2026**:
> Если вам нужна конкурентность — используйте `TaskGroup` / `async let`  
> Если нужна защита состояния — используйте `actor`  
> Если нужна просто фоновая работа — используйте `DispatchQueue.global(qos:)`

### 4. Основные методы добавления задач (dispatch)

| Метод                                | Тип отправки       | Блокирует текущий поток? | Самый частый use-case в 2026 | Примечание |
|--------------------------------------|---------------------|---------------------------|-------------------------------|------------|
| `queue.async { ... }`                | Асинхронная         | Нет                       | Почти всё, кроме UI           | Самый безопасный |
| `queue.sync { ... }`                 | Синхронная          | Да                        | Редко — только внутри actor   | Опасно на main → deadlock |
| `queue.asyncAfter(deadline:)`        | Отложенная асинхронная | Нет                    | Дебонсинг, тосты, анимации    | Заменяется на `Task.sleep` |
| `queue.async(flags: .barrier) { ... }` | Барьерная асинхронная | Нет                  | Безопасная запись в коллекцию | Почти всегда заменяется actor |
| `queue.async(execute: workItem)`     | Асинхронная с отменой | Нет                    | Отмена задач при уходе с экрана | Legacy, лучше Task.cancel |

### 5. Самые популярные шаблоны dispatch в 2026 году

#### Шаблон 1 — Фон → UI (самый частый)

```swift
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

#### Шаблон 2 — Современный эквивалент (Swift Concurrency)

```swift
@MainActor
class UserViewModel: ObservableObject {
    @Published var user: User?
    
    func load() async {
        let fetched = try await fetchUser()
        user = fetched  // безопасный dispatch на main
    }
}
```

#### Шаблон 3 — Барьерная запись (legacy, но всё ещё встречается)

```swift
let queue = DispatchQueue.global(qos: .utility)
var cache: [String: Data] = [:]

func saveToCache(_ data: Data, key: String) {
    queue.async(flags: .barrier) {
        cache[key] = data
    }
}
```

**Лучше в 2026**:

```swift
actor Cache {
    private var storage: [String: Data] = [:]
    
    func save(_ data: Data, key: String) {
        storage[key] = data
    }
}
```

### 6. DispatchQueue vs Swift Concurrency (честное сравнение 2026)

| Характеристика                     | DispatchQueue / GCD                     | Swift Concurrency (Task, actor)          | Что выбрать в 2026 году |
|------------------------------------|------------------------------------------|-------------------------------------------|--------------------------|
| Потокобезопасность                 | Нужно вручную (lock, barrier)            | Встроенная (actor, Sendable)              | Swift Concurrency        |
| Приоритеты                         | QoS                                      | Task priority                             | Оба хороши, Task проще   |
| Отмена задач                       | DispatchWorkItem (сложно)                | Нативная (Task.cancel)                    | Swift Concurrency        |
| UI-обновления                      | DispatchQueue.main.async                 | @MainActor / await MainActor.run          | @MainActor лучше         |
| Читаемость                         | Старый стиль                             | Современный, меньше кода                  | Swift Concurrency        |
| Совместимость с legacy             | Отличная                                 | Требует адаптации                         | GCD для старого кода     |

### 7. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — DispatchQueue уже считается legacy  
- **QoS** — всегда указывайте явно  
- **Не используйте** `sync` в production — почти всегда это ошибка  
- **Для UI** — только `@MainActor` / `MainActor.run` / `DispatchQueue.main.async`  
- **Для тестов** — используйте `XCTestExpectation` + `queue.async` или `Task`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасное использование GCD

**Короткий девиз 2026**:
> «Dispatch Queue — это когда нужно положить задачу в очередь и забыть.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — Task, actor, @MainActor, TaskGroup.  
> Используй GCD только для поддержки старого кода.»

Удачи с быстрым, безопасным и современным многопоточным кодом в Swift! 🚀