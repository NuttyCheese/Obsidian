Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по **`DispatchQueue.global`** в Swift и Grand Central Dispatch (GCD).

### 1. Что такое DispatchQueue.global (реальность 2026 года)

`DispatchQueue.global(qos:)` — это **статический метод**, который возвращает одну из **системных глобальных конкурентных очередей**.

Это **самый часто используемый** способ запустить фоновую задачу в GCD.

**Ключевые факты 2026 года**:

- Все глобальные очереди — **конкурентные** (parallel)  
- Они используют **общий системный пул потоков** (4–64+ потоков в зависимости от устройства)  
- Порядок выполнения задач **не гарантируется**  
- Количество одновременно работающих задач зависит от **QoS**, загруженности CPU и политики iOS  
- В Swift 6+ и строгой проверке concurrency — передача mutable данных в глобальные очереди часто вызывает предупреждения/ошибки

### 2. Доступные QoS (приоритеты) в 2026 году

| QoS                     | Когда использовать (рекомендация Apple 2026) | Пример задач в iOS-приложении 2026 | Относительная скорость выполнения |
|-------------------------|-----------------------------------------------|-------------------------------------|------------------------------------|
| `.userInteractive`      | Анимации, реакция на касание, скролл          | Обновление UI во время жеста, анимации | Самый высокий                      |
| `.userInitiated`        | Задачи, инициированные пользователем          | Загрузка данных для текущего экрана | Очень высокий                      |
| `.default`              | Задачи без явного приоритета                  | Фоновые операции по умолчанию       | Средний                            |
| `.utility`              | Долгие, некритичные задачи                    | Обработка фото, аналитика, индексация | Средний-низкий                     |
| `.background`           | Очень низкий приоритет                        | Резервное копирование, очистка кэша | Самый низкий                       |

**Важно**:  
`.userInitiated` и `.userInteractive` могут **вытеснять** задачи с более низким QoS → это главная причина **Priority Starvation** в 2026 году.

### 3. Самые частые и правильные шаблоны использования в 2026

#### Шаблон 1 — Фон → UI (самый частый паттерн)

```swift
func loadProfile() {
    DispatchQueue.global(qos: .userInitiated).async {
        // Тяжёлая работа: сеть, парсинг, Core Data
        let profile = fetchProfileFromNetwork()
        
        DispatchQueue.main.async {
            // Только UI-обновление
            self.nameLabel.text = profile.name
            self.avatarImageView.image = profile.avatar
            self.tableView.reloadData()
        }
    }
}
```

#### Шаблон 2 — Параллельная обработка с ограничением (чтобы не было Thread Explosion)

```swift
let queue = DispatchQueue.global(qos: .utility)

for imageURL in imageURLs {
    queue.async {
        guard let data = try? Data(contentsOf: imageURL),
              let image = UIImage(data: data) else { return }
        
        DispatchQueue.main.async {
            self.collectionView.insertImage(image)
        }
    }
}
```

**Лучше в 2026** — через `TaskGroup`:

```swift
await withTaskGroup(of: Void.self) { group in
    for url in imageURLs.prefix(8) {  // лимит 8 параллельных загрузок
        group.addTask(priority: .utility) {
            let image = try? await downloadImage(url)
            await MainActor.run {
                self.collectionView.insertImage(image)
            }
        }
    }
}
```

#### Шаблон 3 — Барьерная запись в общую коллекцию

```swift
let queue = DispatchQueue.global(qos: .utility)
var processedResults: [String: Result] = [:]

func processItem(_ item: Item) {
    queue.async(flags: .barrier) {
        let result = heavyProcess(item)
        processedResults[item.id] = result  // безопасно
    }
}
```

**Лучше в 2026** — через `actor`:

```swift
actor ResultStore {
    private var results: [String: Result] = [:]
    
    func store(_ result: Result, for id: String) {
        results[id] = result
    }
}
```

### 4. DispatchQueue.global vs Swift Concurrency (честное сравнение 2026)

| Характеристика                     | DispatchQueue.global(qos:)             | Swift Concurrency (Task, actor)        | Что выбрать в 2026 году |
|------------------------------------|-----------------------------------------|----------------------------------------|--------------------------|
| Потокобезопасность                 | Нужно вручную (lock, barrier)           | Встроенная (actor, Sendable)           | Swift Concurrency        |
| Приоритеты                         | QoS (.userInitiated, .background…)      | Task priority (.userInitiated, .background) | Оба хороши, Task проще |
| Отмена задач                       | DispatchWorkItem (сложно)               | Нативная (Task.cancel())               | Swift Concurrency        |
| UI-обновления                      | DispatchQueue.main.async                | @MainActor / await MainActor.run       | @MainActor лучше         |
| Читаемость                         | Старый стиль, много boilerplate         | Современный, меньше кода               | Swift Concurrency        |
| Совместимость с legacy             | Отличная                                | Требует адаптации                      | GCD для старого кода     |

**Вывод 2026**:
- **Новый код** → **Swift Concurrency** (`Task`, `actor`, `TaskGroup`, `@MainActor`)  
- **Поддержка старого кода** → `DispatchQueue.global(qos:)`  
- `DispatchQueue.concurrent(label:)` — почти никогда не используйте

### 5. Типичные ошибки с DispatchQueue.global в 2026

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| `sync` на глобальной очереди из главного потока | Deadlock или сильное замедление          | Никогда не использовать `sync` на global из main |
| Перегруженная очередь `.userInitiated`      | Priority Starvation фоновых задач        | Использовать правильные QoS |
| Обновление UI без `.main.async`             | Main Thread Violation                    | Всегда `@MainActor` или `DispatchQueue.main.async` |
| Передача mutable данных между задачами      | Data Race                                | Использовать actor / Sendable |
| Создание тысяч задач без ограничения        | Thread Explosion                         | TaskGroup с лимитом concurrency |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — DispatchQueue.global уже считается legacy  
- **QoS** — всегда указывайте явно (`.userInteractive`, `.userInitiated`, `.utility`, `.background`)  
- **Не используйте** `sync` в production — почти всегда это ошибка  
- **Для UI** — только `@MainActor` / `MainActor.run` / `DispatchQueue.main.async`  
- **Для тестов** — используйте `XCTestExpectation` + `queue.async` или `TaskGroup`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасное использование глобальных очередей

**Короткий девиз 2026**:
> «DispatchQueue.global — это когда нужно быстро запустить задачу в фоне.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — Task, actor, @MainActor, TaskGroup.  
> Используй GCD только для поддержки старого кода.»

Удачи с быстрым, безопасным и современным многопоточным кодом в Swift! 🚀