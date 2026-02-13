### 1. Что такое Executor простыми словами

**Executor** — это **абстракция**, которая отвечает за то, **на каком потоке (или в каком контексте)** будет выполняться тело асинхронной функции или метод актора.

В [[Swift Concurrency]] **каждый [[actor]]** (включая `@MainActor`) имеет **свой собственный executor**.

Главные задачи любого executor-а:

- **Гарантировать изоляцию** — в один момент времени только один поток работает с состоянием актора  
- **Переключать контекст** — когда происходит [[await]], [[Swift]] автоматически «перепрыгивает» на нужный executor  
- **Определять поток** — на каком именно потоке (или пуле потоков) будет выполняться код  
- **Обеспечивать последовательность** — методы одного актора никогда не выполняются параллельно

**Коротко и по-человечески**:
> Executor — это «личный планировщик» каждого актора.  
> Он отвечает за вопрос: «Кто и когда может трогать мои данные?»

### 2. Основные виды Executor в 2026 году

| Вид Executor               | На каком потоке работает                       | Глобальный? | Изоляция    | Самый частый use-case 2026      |
| -------------------------- | ---------------------------------------------- | ----------- | ----------- | ------------------------------- |
| **MainActor executor**     | Всегда на **главном потоке** ([[main thread]]) | Да          | Полная      | UI, ViewModel, [[SwiftUI]]      |
| **Обычный actor executor** | На **любом потоке** из глобального пула        | Нет         | Полная      | Данные, сервисы, кэш            |
| **Detached executor**      | На **любом потоке** ([[Task]].detached)        | Нет         | Отсутствует | Фоновые вычисления без изоляции |
| **Custom executor**        | На потоке / очереди, которую вы сами задаёте   | Нет         | Полная      | Редко (низкоуровневый код)      |

**Важно**:
- `Task { ... }` без явной аннотации → **не привязан к конкретному executor** (может быть любой поток)  
- `Task { @MainActor in ... }` → сразу на MainActor executor  
- `Task.detached { ... }` → detached executor (без изоляции)  
- `await actor.method()` → автоматически переключает на executor этого actor-а

### 3. Самые популярные шаблоны 2026 года

#### Шаблон 1 — MainActor executor (UI)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    
    func load() async {
        // Этот код уже на MainActor executor
        let fetched = try? await api.fetchProfile()
        profile = fetched  // безопасно обновляем UI
    }
}
```

#### Шаблон 2 — Обычный actor executor (данные)

```swift
actor UserCache {
    private var users: [UUID: User] = [:]
    
    func store(_ user: User) {
        users[user.id] = user  // безопасно — внутри executor
    }
    
    func get(_ id: UUID) -> User? {
        users[id]
    }
}

let cache = UserCache()

Task {
    let user = User(id: UUID(), name: "Alex")
    await cache.store(user)
    let loaded = await cache.get(user.id)
}
```

#### Шаблон 3 — Detached executor + переключение на MainActor

```swift
Task.detached(priority: .utility) {
    // Этот код на detached executor (любой поток)
    let data = heavyComputation()
    
    await MainActor.run {
        // Переключаемся на MainActor executor
        self.tableView.reloadData(with: data)
    }
    
    await someActor.process(data)  // переключаемся на executor someActor
}
```

#### Шаблон 4 — Явное переключение executor (редко, но мощно)

```swift
actor Database {
    func query() async -> [String] { ... }
}

Task {
    print("До:", Thread.current)
    
    await Database().query()  // переключаемся на executor Database
    
    print("После actor:", Thread.current)
    
    await MainActor.run {
        print("На главном потоке:", Thread.current)
    }
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                                     | Последствия                       | Как избежать                                                               |
| ---------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| Доступ к actor-свойству без `await`                        | Ошибка компиляции                 | Всегда `await actor.property`                                              |
| Вызов actor-метода без `await`                             | Ошибка компиляции                 | Компилятор напомнит                                                        |
| Долгая синхронная работа внутри `@MainActor`               | UI freeze (ANR)                   | Тяжёлое — в `Task.detached` или обычный `actor`                            |
| Думать, что `Task { ... }` внутри actor наследует executor | Нет — новый независимый контекст  | Используй `@_inheritActorContext` (редко) или `Task { @MainActor in ... }` |
| Забыть переключиться на MainActor для UI                   | [[Main Thread Violation]]         | Все UI-обновления — через `@MainActor`                                     |
| Слишком много actor-ов с тяжёлыми задачами                 | Переключение контекста → задержки | Делай actor-ы маленькими                                                   |

### 5. Executor vs другие механизмы изоляции (2026 сравнение)

| Механизм                  | Executor по умолчанию              | Изоляция | Глобальность | Рекомендация 2026 |
|---------------------------|------------------------------------|----------|--------------|-------------------|
| `@MainActor`              | MainActor executor (главный поток) | Полная   | Да           | UI, ViewModel     |
| обычный `actor`           | Собственный executor               | Полная   | Нет          | Данные, сервисы   |
| `Task { ... }`            | Нет (любой поток)                  | Отсутствует | Нет        | Фоновые задачи    |
| `Task.detached { ... }`   | Detached executor                  | Отсутствует | Нет        | Максимальный параллелизм |
| `DispatchQueue`           | Ручная очередь                     | Ручная   | Нет          | Legacy-код        |

### 6. Лучшие практики 2026 года

- **actor** — для любого **изменяемого состояния** в многопотоке  
- **[[@MainActor]]** — для **всего UI** (ViewModel, контроллеры, [[SwiftUI]])  
- **Task.detached** — для **тяжёлых фоновых вычислений** без изоляции  
- **Переключение контекста** — используй `await MainActor.run` или `await actor.method()`  
- **[[nonisolated]]** — для методов, которые не трогают состояние актёра  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит почти все ошибки с executor  
- **Мониторинг** — Instruments → Swift Tasks — смотри переключения контекста и suspension points

**Короткий девиз 2026**:
> «Executor — это «личный секретарь» каждого актора: он решает, кто и когда может работать с его данными.  
> В 2026 году почти всё изменяемое состояние живёт внутри actor-ов со своим executor-ом, а UI — на MainActor executor.»
