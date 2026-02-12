### 1. Что такое Incorrect Locking

**Incorrect Locking** — это **любое некорректное использование механизмов синхронизации** (locks, queues, semaphores, actors и т.д.), которое приводит к одному или нескольким из следующих последствий:

- **[[Deadlock]]** (взаимная блокировка)  
- **[[Data Race]]** (гонка данных)  
- **[[Priority Starvation|Starvation]]** (голодание потоков)  
- **[[Priority inversion]]** (инверсия приоритетов)  
- **Значительная потеря производительности**  
- **Зависание UI** или всего приложения

**Самые частые проявления в iOS-приложениях 2026**:

- Приложение зависает на 100% CPU или 0% CPU (deadlock)  
- Случайные краши с `EXC_BAD_ACCESS` / `SIGSEGV`  
- Неправильные / потерянные данные в @Published / @StateObject  
- UI обновляется с огромной задержкой  
- Тесты падают только в многопоточных сценариях  
- Thread Sanitizer показывает race, но приложение «иногда работает»

### 2. Топ-10 самых частых видов Incorrect Locking в Swift 2026

| №  | Вид ошибки                                      | Последствия                              | Частота в iOS-приложениях | Пример в коде |
|----|--------------------------------------------------|------------------------------------------|----------------------------|---------------|
| 1  | Пропущенный / забытый `unlock()`                | Deadlock / starvation                    | ★★★★★                      | `lock.lock()` без `unlock` |
| 2  | Разный порядок захвата нескольких lock          | Deadlock                                 | ★★★★★                      | lockA → lockB vs lockB → lockA |
| 3  | Долгая работа под lock (network, I/O, heavy calc) | UI freeze, starvation других потоков     | ★★★★☆                      | `lock.lock(); sleep(5); lock.unlock()` |
| 4  | `.sync` на текущей очереди (чаще main)          | Deadlock                                 | ★★★★☆                      | `DispatchQueue.main.sync { ... }` внутри main |
| 5  | Захват lock в неожиданном месте (внутри closure) | Deadlock / race                          | ★★★★☆                      | lock внутри async closure |
| 6  | Использование lock для чтения immutable данных   | Ненужный overhead, снижение производительности | ★★★☆☆                      | `lock.lock(); let x = immutable; lock.unlock()` |
| 7  | Неправильное использование `os_unfair_lock`     | Краш / UB (undefined behavior)           | ★★★☆☆                      | Забытый `os_unfair_lock_unlock` |
| 8  | Циклическая зависимость между акторами           | Deadlock в Swift Concurrency             | ★★★★☆ (Swift 6+)           | actor A ждёт actor B, B ждёт A |
| 9  | Передача mutable состояния между акторами без Sendable | Data Race / компилятор-ошибка в Swift 6  | ★★★★☆                      | `var` передаётся между Task |
| 10 | Использование `NSRecursiveLock` без необходимости | Ненужный overhead, потенциальные deadlock | ★★☆☆☆                      | Рекурсивный lock вместо простого |

### 3. Самые опасные и популярные антипаттерны 2026

#### Антипаттерн 1 — Забытый unlock (самый частый)

```swift
let lock = NSLock()
var counter = 0

func increment() {
    lock.lock()
    counter += 1
    // забыли unlock → deadlock навсегда
}
```

**Правильно**:

```swift
func increment() {
    lock.lock()
    defer { lock.unlock() }  // defer спасает
    counter += 1
}
```

#### Антипаттерн 2 — Разный порядок захвата lock

```swift
// Поток 1
lockA.lock()
lockB.lock()
// ...

// Поток 2
lockB.lock()
lockA.lock()
// ...
```

**Правильно** — всегда один порядок:

```swift
// Всегда A → B
lockA.lock()
lockB.lock()
// ...
lockB.unlock()
lockA.unlock()
```

#### Антипаттерн 3 — Долгая работа под lock

```swift
let lock = NSLock()
var cache: [String: Data] = [:]

func getImage(url: String) -> Data? {
    lock.lock()
    if let cached = cache[url] {
        lock.unlock()
        return cached
    }
    
    // ← Ошибка! Долгая операция под lock
    let data = try! Data(contentsOf: URL(string: url)!)
    cache[url] = data
    lock.unlock()
    return data
}
```

**Правильно** — минимизировать критическую секцию:

```swift
func getImage(url: String) -> Data? {
    lock.lock()
    if let cached = cache[url] {
        lock.unlock()
        return cached
    }
    lock.unlock()
    
    let data = try? Data(contentsOf: URL(string: url)!)
    
    lock.lock()
    cache[url] = data
    lock.unlock()
    
    return data
}
```

### 4. Все современные способы защиты от Incorrect Locking (2026)

| Способ                              | Защита от deadlock | Защита от data race | Скорость | Сложность | Рекомендация 2026 |
|-------------------------------------|--------------------|----------------------|----------|-----------|-------------------|
| **actor**                           | ★★★★★              | ★★★★★                | ★★★★☆    | ★★★☆☆     | Основной выбор    |
| **@MainActor / @globalActor**       | ★★★★★              | ★★★★★                | ★★★★☆    | ★★☆☆☆     | UI и сервисы      |
| **Sendable + strict concurrency**   | ★★★★★              | ★★★★★                | ★★★★★    | ★★★★☆     | Обязательно в Swift 6 |
| **os_unfair_lock + defer**          | ★★★★☆              | ★★★★☆                | ★★★★★    | ★★★★☆     | Высокопроизводительные места |
| **DispatchQueue (serial)**          | ★★★★☆              | ★★★★☆                | ★★★★☆    | ★★☆☆☆     | Legacy-код        |
| **Atomic** (swift-atomics)          | ★★★★☆              | ★★★★★                | ★★★★★    | ★★★☆☆     | Простые примитивы |
| **Value types + immutability**      | ★★★★★              | ★★★★★                | ★★★★★    | ★★☆☆☆     | Идеальный вариант |

### 5. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### actor — золотой стандарт

```swift
actor SafeCache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func store(_ value: Value, for key: Key) {
        storage[key] = value
    }
    
    func fetch(for key: Key) -> Value? {
        storage[key]
    }
}

let cache = SafeCache<String, Data>()

Task.detached {
    let data = try await downloadData()
    await cache.store(data, for: "profile")
}
```

#### @MainActor для UI и ViewModel

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    
    func load() async {
        user = try? await fetchUser()  // безопасно
    }
}
```

### 6. Как найти Incorrect Locking и Deadlock в 2026

| Инструмент / Способ               | Что ловит                              | Где включается                  | Эффективность |
|-----------------------------------|----------------------------------------|----------------------------------|---------------|
| **Thread Sanitizer**              | Data Race, некоторые deadlock          | Xcode → Scheme → Diagnostics     | ★★★★☆         |
| **Instruments → Thread Analyzer** | Deadlock, долгие блокировки            | Instruments                      | ★★★★★         |
| **Xcode Debugger + Threads**      | Зависшие потоки, порядок lock          | Debug navigator → Threads        | ★★★★☆         |
| **os_signpost + Instruments**     | Долгие критические секции              | Instruments → Points of Interest | ★★★★☆         |
| **Swift 6 Strict Concurrency**    | Передача mutable данных между акторами | Build Settings → Swift Compiler  | ★★★★★         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на [[Swift]] 6** — strict concurrency checking ловит большинство потенциальных [[deadlock]] и [[data race]]
- **[[actor]]** — основной способ хранения изменяемого состояния  
- **[[@MainActor]]** — всё, что касается UI и [[SwiftUI]]  
- **[[Sendable]]** — всё, что передаётся между акторами  
- **Никогда** не вызывайте `.sync` на текущей очереди (особенно main)  
- **Фиксированный порядок захвата lock** — если всё ещё используете [[NSLock]]/os_unfair_lock  
- **Минимизируйте критическую секцию** — захватывайте lock только на время изменения данных  
- **[[defer]] { unlock() }** — спасает от забытого unlock  
- **Для тестов** — используйте [[actor]] + [[XCTest]] с [[await]]
- **Для legacy-кода** — постепенно оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Incorrect Locking — это когда замки есть, но они не спасают, а убивают.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> NSLock, sync, глобальные var — это уже вчерашний день.»
