### 1. Что значит «dispatching tasks» в [[GCD]]

**Dispatching** (отправка / диспетчеризация задач) — это процесс **помещения блока кода** (замыкания / [[closure]]) в очередь диспетчера (`DispatchQueue`), чтобы система GCD сама решила:

- когда именно выполнить задачу  
- на каком потоке (или в каком пуле потоков)  
- синхронно или асинхронно  
- с каким приоритетом (QoS)

**Коротко и по-человечески**:
> «dispatch» = «положить задачу в очередь и сказать системе: делай когда сможешь, я не буду ждать (или буду, если sync)»

Это **основной способ** многопоточности в iOS/macOS с 2009 года и до сих пор (2026), хотя в новом коде его всё чаще заменяют Swift Concurrency.

### 2. Основные способы dispatching (отправки) задач

| Метод dispatch                     | Тип отправки       | Блокирует текущий поток? | Самый частый use-case в 2026 | Примечание / опасность |
|------------------------------------|---------------------|---------------------------|-------------------------------|-------------------------|
| `queue.async { ... }`              | Асинхронная         | Нет                       | Почти всё, кроме UI           | Самый безопасный и частый |
| `queue.sync { ... }`               | Синхронная          | Да                        | Редко — только внутри actor   | Опасно на main → deadlock |
| `queue.asyncAfter(deadline:)`      | Отложенная асинхронная | Нет                    | Дебонсинг, тосты, анимации    | Заменяется на `Task.sleep` |
| `queue.async(flags: .barrier) { ... }` | Барьерная асинхронная | Нет                  | Безопасная запись в коллекцию | Почти всегда заменяется actor |
| `queue.async(execute: workItem)`   | Асинхронная с отменой | Нет                    | Отмена задач при уходе с экрана | Legacy, лучше Task.cancel |

### 3. Самые популярные шаблоны dispatching в 2026 году

#### Шаблон 1 — Фон → UI (самый частый паттерн)

```swift
func loadUserProfile() {
    DispatchQueue.global(qos: .userInitiated).async {
        // Тяжёлая работа: сеть, парсинг, Core Data
        let profile = fetchProfileFromNetwork()
        
        DispatchQueue.main.async {
            // Только UI-обновление
            self.avatarImageView.image = profile.avatar
            self.nameLabel.text = profile.name
            self.tableView.reloadData()
        }
    }
}
```

#### Шаблон 2 — Современный эквивалент (Swift Concurrency 2026)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    
    func load() async {
        let fetched = try await fetchProfile()
        profile = fetched  // безопасный dispatch на main
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

### 4. Dispatch vs Swift Concurrency (честное сравнение 2026)

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
- `DispatchQueue.concurrent(label:)` — почти никогда не создавайте

### 5. Типичные ошибки dispatching в 2026

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| `sync` на текущей очереди                   | Deadlock                                 | Никогда не делать `.sync` внутри той же очереди |
| `sync` на `.main` из главного потока        | Deadlock / зависание UI                  | Только `async` на main |
| Обновление UI без `.main.async`             | Main Thread Violation                    | Всегда `@MainActor` или `DispatchQueue.main.async` |
| Передача mutable данных между задачами      | Data Race                                | Использовать actor / Sendable |
| Перегруженная очередь `.userInitiated`      | Priority Starvation                      | Ограничивать concurrency (TaskGroup) |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — DispatchQueue уже считается legacy  
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
> Используй GCD только для поддержки старого кода.»

Удачи с быстрым, безопасным и современным многопоточным кодом в Swift! 🚀