Вот **полное, подробное и максимально актуальное** (на февраль 2026 года) руководство по протоколу **`Sendable`** в Swift — включая все нюансы Swift 6 и строгой конкурентности.

### 1. Что такое Sendable и зачем он нужен

**`Sendable`** — это **маркерный протокол** (marker protocol), который означает:

> «Этот тип **безопасно передавать между разными задачами (tasks), актёрами и executor’ами** без риска гонок данных (data races)».

В Swift Concurrency любое значение, которое **передаётся** между разными изолированными контекстами (actor → actor, Task → Task, detached task → main и т.д.), **обязательно** должно быть `Sendable`.

**Ключевые моменты 2026 года**:

- `Sendable` — **не обычный протокол** с методами, а **маркер** (как `Sendable`, `~Copyable`, `~Escapable`)  
- Компилятор **автоматически проверяет** conformance в строгом режиме  
- В Swift 6 с полной проверкой (`strict concurrency checking = complete`) почти **все** типы, которые передаются между задачами, **должны** быть `Sendable` — иначе ошибка компиляции  
- `actor` → **автоматически** `Sendable`  
- `struct` / `enum` / `tuple` → автоматически `Sendable`, если все их поля тоже `Sendable`  
- `class` → **никогда** не `Sendable` автоматически — нужно `@unchecked Sendable` или полная потокобезопасность

**Коротко**:
> Sendable = «этот объект можно безопасно копировать / передавать между потоками / задачами / актёрами без синхронизации».

### 2. Автоматический и ручной conformance (таблица 2026)

| Тип данных                        | Автоматически Sendable? | Условия автоматического conformance | Как сделать Sendable вручную |
|-----------------------------------|--------------------------|---------------------------------------|-------------------------------|
| `struct` / `enum` / `tuple`       | Да                       | Все поля — `Sendable`                 | Не нужно (автоматически)      |
| `actor`                           | Да                       | Всегда                                | Не нужно                      |
| `final class` без `var`           | Да                       | Только `let` свойства, все `Sendable` | `: Sendable` (автоматически)  |
| `final class` с `var`             | Нет                      | —                                     | `@unchecked Sendable` + синхронизация |
| `class` (не final)                | Нет                      | —                                     | `@unchecked Sendable` (опасно) |
| `AnyObject` / протоколы           | Нет                      | —                                     | `@unchecked Sendable` или `Sendable & AnyObject` |
| `UnsafeMutablePointer` и т.п.     | Нет                      | —                                     | `@unchecked Sendable` (очень опасно) |

### 3. Самые популярные и правильные шаблоны 2026 года

#### Шаблон 1 — Простая структура (автоматический Sendable)

```swift
struct User: Sendable {
    let id: UUID
    let name: String
    let age: Int
}

Task.detached {
    let user = User(id: UUID(), name: "Alex", age: 30)
    print(user.name)  // безопасно передано в detached task
}
```

#### Шаблон 2 — final class без мутации (Sendable)

```swift
final class ConstantConfig: Sendable {
    let apiKey: String
    let endpoint: URL
    
    init(apiKey: String, endpoint: URL) {
        self.apiKey = apiKey
        self.endpoint = endpoint
    }
}

Task {
    let config = ConstantConfig(apiKey: "xyz", endpoint: URL(string: "https://api.com")!)
    await someActor.use(config)  // безопасно
}
```

#### Шаблон 3 — @unchecked Sendable (самый оправданный случай)

```swift
final class ThreadSafeCache<Key: Hashable, Value>: @unchecked Sendable {
    private let queue = DispatchQueue(label: "cache", attributes: .concurrent)
    private var storage: [Key: Value] = [:]
    
    func get(_ key: Key) -> Value? {
        queue.sync { storage[key] }
    }
    
    func set(_ value: Value, for key: Key) {
        queue.async(flags: .barrier) {
            self.storage[key] = value
        }
    }
}
```

Здесь `@unchecked Sendable` оправдан — **вся мутация** защищена concurrent DispatchQueue.

#### Шаблон 4 — actor (автоматически Sendable)

```swift
actor UserRepository {
    private var users: [UUID: User] = [:]
    
    func save(_ user: User) {
        users[user.id] = user
    }
}

let repo = UserRepository()

Task.detached {
    let user = User(id: UUID(), name: "Bob")
    await repo.save(user)  // actor автоматически Sendable
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `: Sendable` на структуре с non-Sendable полями | Ошибка компиляции                        | Все поля должны быть Sendable |
| Поставить `@unchecked Sendable` на класс с открытыми `var` | Гонки данных в runtime                   | Всё mutable — через lock / actor / serial queue |
| Захватить non-Sendable класс в `@Sendable` замыкании | Ошибка компиляции в Swift 6              | Сделать класс `@unchecked Sendable` или использовать actor |
| Думать, что `@unchecked Sendable` делает класс потокобезопасным | Ложное чувство безопасности              | Это **только отключение проверки** — безопасность на тебе |
| Использовать `class` вместо `actor` для состояния | Гонка данных                                 | Всё изменяемое состояние → actor |

### 5. Sendable vs другие механизмы защиты данных (2026 сравнение)

| Механизм                  | Проверка компилятором | Потокобезопасность | Ответственность | Рекомендация 2026 | Когда использовать |
|---------------------------|------------------------|---------------------|------------------|-------------------|---------------------|
| `Sendable` (автоматически) | Полная                 | Полная              | Компилятор + программист | Основной выбор    | Структуры, enum, final-классы без var |
| `@unchecked Sendable`     | Отключена              | Ручная              | Полностью на программисте | Только исключение | Legacy-классы, обёртки |
| `actor`                   | Полная                 | Полная встроенная   | Компилятор       | Самый безопасный  | Любое изменяемое состояние |
| `@MainActor`              | Полная                 | Полная (main thread)| Компилятор       | Для UI            | ViewModel, UI-логика |
| `nonisolated(unsafe)`     | Отключена              | Отсутствует          | Полностью на программисте | Редко             | Низкоуровневый код |

**Вывод 2026**:
- `Sendable` — **фундамент** безопасной передачи данных в конкурентном коде  
- `actor` — **лучший** способ хранения изменяемого состояния  
- `@unchecked Sendable` — **последний рубеж**, когда ничего другого не остаётся  
- В идеальном проекте 2026 года `@unchecked Sendable` должен быть **минимальным**

### 6. Лучшие практики 2026 года

- **Делай все модели `Sendable`** — структуры, enum, DTO  
- **final class без var** — просто `: Sendable`  
- **final class с var** — `@unchecked Sendable` + внутренняя синхронизация (очень осторожно)  
- **Всё изменяемое состояние** — храни в `actor`  
- **Замыкания** — всегда `@Sendable` в `Task`, `async let`, callback  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит почти все проблемы с Sendable  
- **Тестирование** — Thread Sanitizer + Instruments на гонки данных  
- **Документируй** — пиши комментарий «@unchecked Sendable — синхронизация через X»

**Короткий девиз 2026**:
> «Sendable — это обещание: «этот тип можно безопасно передать в другую задачу / актёр».  
> В 2026 году почти всё, что передаётся между задачами, **обязательно** должно быть Sendable — иначе Swift 6 не скомпилирует код.»

Удачи с безопасной, проверяемой и современной конкурентностью в Swift! 🛡️