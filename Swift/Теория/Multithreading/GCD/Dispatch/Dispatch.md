### 1. Что значит «dispatch» в GCD

Слово **dispatch** в Grand Central Dispatch буквально означает:

> «отправка», «диспетчеризация», «планирование», «распределение» задачи на выполнение

В GCD dispatch — это **процесс помещения задачи** (блока кода, замыкания) в очередь (`DispatchQueue`), чтобы система сама решила:

- когда именно её выполнить  
- на каком потоке  
- с каким приоритетом  
- параллельно или последовательно

**Коротко и по-человечески**:
> «dispatch» = «положить задачу в очередь и сказать системе: делай когда сможешь, я не буду ждать»

### 2. Основные сущности, где используется dispatch (2026 актуально)

| Сущность                          | Что это                                      | Как dispatch связан                              | Самый частый метод dispatch |
|-----------------------------------|----------------------------------------------|---------------------------------------------------|-----------------------------|
| DispatchQueue                     | Очередь задач (serial / concurrent / main / global) | Основной объект, куда отправляют (dispatch) задачи | `async`, `sync`, `asyncAfter` |
| DispatchQueue.main                | Главная очередь (main thread)                | dispatch → UI-обновления                          | `main.async { ... }`        |
| DispatchQueue.global(qos:)        | Системные конкурентные очереди               | dispatch → фоновая работа                         | `global(qos:).async { ... }` |
| DispatchWorkItem                  | Объект-задача с возможностью отмены          | dispatch → управляемая, отменяемая задача         | `queue.async(execute: workItem)` |
| DispatchGroup                     | Группа задач для ожидания завершения         | dispatch → отслеживание нескольких задач          | `group.enter()`, `group.leave()` |
| DispatchBarrier                   | Барьерная задача в конкурентной очереди     | dispatch → безопасная запись при параллельном чтении | `async(flags: .barrier)`    |

### 3. Основные способы dispatch (отправки) задач

| Метод dispatch                     | Тип отправки       | Блокирует текущий поток? | Где чаще всего используют в 2026 | Примечание |
|------------------------------------|---------------------|---------------------------|-----------------------------------|------------|
| `queue.async { ... }`              | Асинхронная         | Нет                       | Почти везде                       | Самый безопасный и частый |
| `queue.sync { ... }`               | Синхронная          | Да                        | Редко, только внутри actor или background | Опасно на main → deadlock |
| `queue.asyncAfter(deadline:)`      | Отложенная асинхронная | Нет                    | Дебонсинг, тосты, анимации        | Заменяется на `Task.sleep` |
| `queue.async(flags: .barrier) { ... }` | Барьерная асинхронная | Нет                  | Безопасная запись в коллекцию     | Почти всегда заменяется actor |
| `queue.async(execute: workItem)`   | Асинхронная с отменой | Нет                    | Отмена задач при уходе с экрана   | Legacy, лучше Task.cancel |

### 4. Самые популярные шаблоны dispatch в 2026 году

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

#### Шаблон 2 — Современный эквивалент (Swift Concurrency 2026)

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
let queue = DispatchQueue(label: "com.myapp.cache", attributes: .concurrent)
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

### 5. DispatchQueue vs Swift Concurrency (честное сравнение 2026)

| Характеристика                     | DispatchQueue / GCD                     | Swift Concurrency (Task, actor)          | Что выбрать в 2026 году |
|------------------------------------|------------------------------------------|-------------------------------------------|--------------------------|
| Потокобезопасность                 | Нужно вручную (lock, barrier)            | Встроенная (actor, Sendable)              | Swift Concurrency        |
| Приоритеты                         | QoS                                      | Task priority                             | Оба хороши, Task проще   |
| Отмена задач                       | DispatchWorkItem (сложно)                | Нативная (Task.cancel)                    | Swift Concurrency        |
| UI-обновления                      | DispatchQueue.main.async                 | @MainActor / await MainActor.run          | @MainActor лучше         |
| Читаемость                         | Старый стиль                             | Современный, меньше кода                  | Swift Concurrency        |
| Совместимость с legacy             | Отличная                                 | Требует адаптации                         | GCD для старого кода     |

**Вывод 2026**:
- **Новый код** → **Swift Concurrency** (`Task`, `actor`, `TaskGroup`, `@MainActor`)  
- **Поддержка старого кода** → GCD (`DispatchQueue.global`, `main.async`)  
- `DispatchQueue.concurrent(label:)` — почти никогда не используйте

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — GCD уже считается legacy  
- **QoS** — всегда указывайте явно  
- **Не используйте** `sync` в production — почти всегда это ошибка  
- **Для UI** — только `@MainActor` / `MainActor.run` / `DispatchQueue.main.async`  
- **Для тестов** — используйте `XCTestExpectation` + `queue.async` или `Task`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасное использование GCD

**Короткий девиз 2026**:
> «dispatch — это когда ты говоришь системе: «Возьми эту задачу и сделай её когда-нибудь».  
> В 2026 году это уже legacy-слово.  
> Новый стандарт — Task, actor, @MainActor, TaskGroup.  
> Если ты всё ещё пишешь dispatch в новом коде — спроси себя: «А точно ли это нужно?»»

Удачи с быстрым, безопасным и современным многопоточным кодом в Swift! 🚀