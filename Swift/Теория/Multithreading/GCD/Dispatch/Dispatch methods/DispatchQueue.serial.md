Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по созданию **сериальных очередей** в Swift и Grand Central Dispatch (GCD).

### 1. Сериальная очередь в GCD — что это такое

**Сериальная очередь** (serial queue) — это очередь, в которой задачи выполняются **строго по одной**, в **порядке их добавления** (FIFO — First In, First Out).

- Только **одна задача** может выполняться в любой момент времени.
- Следующая задача **не начинается**, пока предыдущая **не завершится полностью**.
- Все задачи выполняются в **одном и том же потоке** (но этот поток может меняться между разными вызовами, если очередь не привязана явно).

**Главные цели сериальных очередей в 2026 году**:

- Защита **изменяемого состояния** (mutable state) от **race condition**  
- Последовательный доступ к ресурсам (файлы, база данных, кэш, UserDefaults)  
- Гарантированный порядок выполнения операций  
- Простая альтернатива `actor` в legacy-коде или при работе с GCD-библиотеками

### 2. Как правильно создавать сериальную очередь

```swift
// Самый распространённый и рекомендуемый способ (2026)
let serialQueue = DispatchQueue(label: "com.example.serialQueue")

// С явным указанием QoS (очень рекомендуется)
let serialQueue = DispatchQueue(
    label: "com.example.serialQueue",
    qos: .utility
)

// С атрибутами (почти никогда не нужен в 2026)
let serialQueue = DispatchQueue(
    label: "com.example.serialQueue",
    qos: .background,
    attributes: [],           // пустой массив = serial
    autoreleaseFrequency: .inherit,
    target: nil
)
```

**Важно**:  
Метод `DispatchQueue.serial(label:)` **не существует**.  
Это **миф** и **ошибка** в старых статьях и некоторых туториалах.

**Правильный способ** — просто `DispatchQueue(label:)` без атрибута `.concurrent`.

### 3. Сравнение сериальной и конкурентной очереди

| Характеристика                  | Сериальная очередь (`DispatchQueue(label:)`) | Конкурентная очередь (`attributes: .concurrent`) | Глобальная очередь (`global(qos:)`) |
|---------------------------------|-----------------------------------------------|---------------------------------------------------|-------------------------------------|
| Параллелизм                     | 1 задача за раз                               | Несколько задач параллельно                       | Несколько задач параллельно         |
| Порядок выполнения              | Строго по порядку добавления                  | Не гарантирован                                   | Не гарантирован                     |
| Поток выполнения                | Один и тот же (может меняться)                | Пул потоков                                       | Пул потоков                         |
| Защита от race condition        | Да (если все мутации через эту очередь)       | Нет (нужен barrier)                               | Нет (нужен barrier)                 |
| Лучший use-case в 2026          | Защита mutable state, последовательность      | Редко (почти всегда actor лучше)                  | Фоновые независимые задачи          |
| Рекомендация Apple 2026         | Да, для legacy и простых случаев              | Почти не рекомендуется                            | Да, для простых фоновых задач       |

### 4. Самые популярные и правильные шаблоны 2026 года

#### Шаблон 1 — Защита общего mutable состояния

```swift
let databaseQueue = DispatchQueue(label: "com.myapp.database", qos: .utility)

var cachedUsers: [User] = []

func addUser(_ user: User) {
    databaseQueue.async {
        cachedUsers.append(user)
        // сохранение в Core Data / Realm / UserDefaults
        saveToDisk(users: cachedUsers)
    }
}

func getUsers() -> [User] {
    databaseQueue.sync { cachedUsers }  // безопасное чтение
}
```

#### Шаблон 2 — Последовательные сетевые запросы

```swift
let authQueue = DispatchQueue(label: "com.myapp.auth", qos: .userInitiated)

func login(username: String, password: String) {
    authQueue.async {
        let token = try? loginSync(username: username, password: password)
        
        authQueue.async {
            saveToken(token)
            
            DispatchQueue.main.async {
                // UI-обновление
                self.showMainScreen()
            }
        }
    }
}
```

#### Шаблон 3 — Миграция базы данных при запуске

```swift
let migrationQueue = DispatchQueue(label: "com.myapp.migration", qos: .utility)

func migrateDatabaseIfNeeded(completion: @escaping () -> Void) {
    migrationQueue.async {
        // Миграция Core Data / Realm / SQLite
        performMigration()
        
        DispatchQueue.main.async {
            completion()
        }
    }
}
```

### 5. Типичные ошибки с сериальными очередями в 2026

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| `sync` на текущей очереди                   | Deadlock                                 | Никогда не делать `.sync` внутри той же очереди |
| `sync` на сериальной очереди из главного потока | Deadlock / зависание UI                  | Только `async` или `@MainActor` |
| Перегруженная сериальная очередь            | Задержка всех задач                      | Разделять очереди по смыслу (database, network, analytics) |
| Забыть указать QoS                          | Низкий приоритет по умолчанию            | Всегда задавать `.userInitiated`, `.utility` и т.д. |
| Передача mutable данных без синхронизации   | Data Race                                | Всё mutable состояние — только через очередь / actor |

### 6. DispatchQueue (serial) vs Swift Concurrency (2026 сравнение)

| Характеристика                     | DispatchQueue (serial)                  | actor / Task                            | Что выбрать в 2026 |
|------------------------------------|-----------------------------------------|-----------------------------------------|---------------------|
| Потокобезопасность                 | Ручная (очередь)                        | Встроенная                              | actor               |
| Читаемость                         | Старый стиль                            | Современный                             | actor / Task        |
| Отмена задач                       | Сложно                                  | Нативная                                | Swift Concurrency   |
| Strict Concurrency Checking        | Частично                                | Полностью                               | Swift Concurrency   |
| Производительность                 | Хорошая                                 | Хорошая / лучше в большинстве случаев   | actor / Task        |
| Совместимость с legacy             | Отличная                                | Требует адаптации                       | GCD для старого кода |

**Вывод 2026**:  
- **Новый код** → **actor** для состояния, **Task** / **TaskGroup** для асинхронных операций  
- **Поддержка старого кода** → сериальные `DispatchQueue`  
- `DispatchQueue.concurrent(label:)` — почти никогда не используйте

### 7. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — сериальные очереди уже считаются legacy  
- **QoS** — всегда указывайте явно  
- **Названия очередей** — используйте reverse-DNS (`com.company.module.operation`)  
- **Не используйте** `sync` в production — почти всегда это ошибка  
- **Для UI** — только `@MainActor` / `MainActor.run` / `DispatchQueue.main.async`  
- **Для тестов** — используйте `XCTestExpectation` + `queue.async` или `Task`  
- **Мониторинг** — добавляйте `os_signpost` для отслеживания длительности задач в Instruments  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасное использование очередей

**Короткий девиз 2026**:
> «Сериальная очередь — это когда нужно гарантировать порядок и безопасность доступа к данным.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — actor + Task + @MainActor + strict concurrency.  
> Если ты всё ещё пишешь DispatchQueue в новом коде — спроси себя: «А точно ли это нужно?»»

Удачи с чистым, безопасным и современным многопоточным кодом в Swift! 🚀