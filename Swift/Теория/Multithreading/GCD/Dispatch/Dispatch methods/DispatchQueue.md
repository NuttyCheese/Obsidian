### 1. Что такое DispatchQueue (коротко и честно)

`DispatchQueue` — это **очередь задач** в системе Grand Central Dispatch ([[GCD]]).

Она решает три главные задачи:

- **В какую очередь** положить задачу  
- **В каком порядке** её выполнять  
- **На каком потоке** (или пуле потоков) запускать

Типы очередей в 2026 году:

| Тип очереди                       | Параллелизм         | Порядок выполнения | Основное назначение в 2026 году                     | Пример создания                             |
| --------------------------------- | ------------------- | ------------------ | --------------------------------------------------- | ------------------------------------------- |
| **Сериальная** ([[serial]])       | Один поток          | Строго по очереди  | Защита mutable состояния, последовательные операции | `DispatchQueue(label: "com.my.serial")`     |
| **Конкурентная** ([[concurrent]]) | Пул потоков (4–64+) | Произвольный       | Параллельные независимые задачи                     | `DispatchQueue.global(qos: .userInitiated)` |
| **Глобальная** (global)           | Конкурентная        | Произвольный       | Быстрый доступ к системным пулам                    | `DispatchQueue.global(qos: .background)`    |
| **Главная** (main)                | Главный поток       | Строго по очереди  | Всё, что связано с UI и [[SwiftUI]]                 | `DispatchQueue.main`                        |

### 2. Основные методы DispatchQueue (самые используемые в 2026)

| Метод                                | Что делает                                              | Блокирует текущий поток? | Самый частый use-case в 2026 |
|--------------------------------------|----------------------------------------------------------|---------------------------|-------------------------------|
| `async { ... }`                      | Добавить задачу асинхронно                              | Нет                       | Почти всё, кроме UI           |
| `sync { ... }`                       | Добавить задачу синхронно (ждать завершения)            | Да                        | Редко, только внутри actor или background |
| `asyncAfter(deadline:execute:)`      | Отложить задачу на время                                | Нет                       | Дебounce, throttle, delayed actions |
| `async(flags:execute:)`              | С флагами (`.barrier`, `.enforceQoS` и т.д.)            | Нет                       | Барьерная запись в коллекцию |
| `sync(flags:execute:)`               | Синхронно с флагами                                      | Да                        | Редко, опасно на main         |

### 3. Самые важные шаблоны использования в 2026 году

#### Шаблон 1 — Фон → UI (самый частый)

```swift
func loadUserData() {
    DispatchQueue.global(qos: .userInitiated).async {
        // Тяжёлая работа: сеть, парсинг, Core Data
        let user = fetchUserFromNetwork()
        
        DispatchQueue.main.async {
            // Только UI-обновление
            self.nameLabel.text = user.name
            self.tableView.reloadData()
        }
    }
}
```

#### Шаблон 2 — Барьерная запись в коллекцию (очень актуально)

```swift
let queue = DispatchQueue(label: "com.my.concurrent.array", attributes: .concurrent)
var items: [String] = []

func addItem(_ item: String) {
    queue.async(flags: .barrier) {
        items.append(item)  // только один поток может писать
    }
}

func readItems() -> [String] {
    queue.sync { items }  // чтение безопасно
}
```

#### Шаблон 3 — Ожидание нескольких задач (альтернатива [[DispatchGroup]])

```swift
let queue = DispatchQueue.global(qos: .utility)
let group = DispatchGroup()

queue.async(group: group) {
    // задача 1
}

queue.async(group: group) {
    // задача 2
}

group.notify(queue: .main) {
    // все задачи закончились → обновляем UI
}
```

### 4. DispatchQueue vs Swift Concurrency (сравнение 2026)

| Характеристика             | DispatchQueue / GCD                      | Swift Concurrency ([[Task]], [[actor]])        | Что выбрать в 2026 году   |
| -------------------------- | ---------------------------------------- | ---------------------------------------------- | ------------------------- |
| Потокобезопасность         | Нужно вручную (lock, queue)              | Встроенная (actor, [[Sendable]])               | Swift Concurrency         |
| Приоритеты                 | [[QoS]] (.userInteractive, .background…) | Task priority (.userInitiated, .background)    | Оба хороши, но Task проще |
| Ожидание завершения        | [[DispatchGroup]] / wait                 | [[TaskGroup]] / async let / withTaskGroup      | TaskGroup лучше           |
| Отмена задач               | Сложно ([[DispatchWorkItem]])            | Нативная (Task.cancel(), withTaskCancellation) | Swift Concurrency         |
| UI-обновления              | DispatchQueue.main.async                 | [[@MainActor]] / await MainActor.run           | @MainActor лучше          |
| Читаемость и современность | Старый стиль, много boilerplate          | Современный, меньше кода                       | Swift Concurrency         |
| Совместимость с legacy     | Отличная                                 | Требует адаптации                              | GCD для старого кода      |

**Вывод 2026 года**:
- **Новый код** → **Swift Concurrency** (`Task`, `actor`, `TaskGroup`, `@MainActor`)  
- **Поддержка старого кода** → GCD + DispatchQueue  
- **Смешанный код** → постепенно мигрировать на Concurrency, оставляя GCD только для совместимости

### 5. Типичные ошибки с DispatchQueue в 2026

| Ошибка                                       | Последствия                   | Как избежать                                    |
| -------------------------------------------- | ----------------------------- | ----------------------------------------------- |
| `sync` на текущей очереди                    | [[Deadlock]]                  | Никогда не делать `.sync` на своей очереди      |
| `sync` на `.main` из главного потока         | Deadlock / зависание UI       | Только `async` на main                          |
| Перегруженная глобальная очередь             | [[Priority Starvation]]       | Создавать свои очереди с нужным QoS             |
| Забыть `enter()` / `leave()` в DispatchGroup | `notify` никогда не сработает | Использовать `defer { leave() }`                |
| Использование `sync` в production            | UI freeze, ANR                | Только `async` + `@MainActor` / `MainActor.run` |
| Передача mutable данных между задачами       | [[Data Race]]                 | Использовать actor / Sendable                   |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — DispatchQueue уже считается legacy  
- **QoS** — всегда указывайте явно (`.userInteractive`, `.userInitiated`, `.utility`, `.background`)  
- **Свои очереди** — вместо `global()` создавайте именованные очереди  
- **Барьер** — используйте `async(flags: .barrier)` для конкурентной записи  
- **defer** — спасает от забытого `leave()` или `unlock()`  
- **Не используйте** `sync` в production — почти всегда это ошибка  
- **Для UI** — только `@MainActor` / `MainActor.run` / `DispatchQueue.main.async`  
- **Для тестов** — используйте `XCTestExpectation` + `group.notify` или `TaskGroup`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач

**Короткий девиз 2026**:
> «DispatchQueue — это когда нужно быстро и надёжно запустить задачу в фоне или на главном потоке.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — Task, actor, @MainActor, TaskGroup.  
> Используй GCD только для поддержки старого кода.»
