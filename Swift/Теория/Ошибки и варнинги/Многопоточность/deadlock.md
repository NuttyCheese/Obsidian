**Deadlock** — это ситуация, когда **два или более потока** (или актора) **ждут друг друга бесконечно**, и **ни один из них не может продолжить выполнение**.

Классическая аналогия:  
Два человека стоят в дверях и оба ждут, пока другой пропустит первым → никто не проходит.

В программировании это почти всегда связано с **блокировками** (locks, queues, semaphores, actors).

**Самые частые симптомы в [[iOS]]-приложении**:

- Приложение **полностью зависает** (UI не реагирует)  
- Поток(и) **не продолжают выполнение** (видно в отладчике)  
- Нет ошибок или исключений — просто молчаливое зависание  
- Часто проявляется **только на реальных устройствах** (симулятор может не показать)  
- В логах Thread Sanitizer / Instruments — **deadlock detected**

### 2. Классические сценарии Deadlock в Swift 2026

#### Сценарий 1 — Два [[NSLock]] / os_unfair_lock в разном порядке (самый классический)

```swift
let lockA = NSLock()
let lockB = NSLock()

// Поток 1
DispatchQueue.global().async {
    lockA.lock()
    print("Поток 1 взял A")
    Thread.sleep(forTimeInterval: 0.1) // имитация работы
    lockB.lock()           // ← ждёт B
    print("Поток 1 взял B")
    lockB.unlock()
    lockA.unlock()
}

// Поток 2
DispatchQueue.global().async {
    lockB.lock()
    print("Поток 2 взял B")
    Thread.sleep(forTimeInterval: 0.1)
    lockA.lock()           // ← ждёт A
    print("Поток 2 взял A")
    lockA.unlock()
    lockB.unlock()
}
```

**Результат**: зависание (deadlock)  
Поток 1 держит A и ждёт B  
Поток 2 держит B и ждёт A  
**Никто не может продолжить**

#### Сценарий 2 — Deadlock на главном потоке с DispatchQueue.main.sync

```swift
DispatchQueue.main.sync {
    print("На главном потоке")
    
    DispatchQueue.main.sync {   // ← deadlock!
        print("Вложенный sync на main")
    }
}
```

**Почему**:  
Главный поток уже выполняет внешний `sync` и ждёт завершения вложенного `sync`.  
Вложенный `sync` не может выполниться, потому что главный поток занят ожиданием.

**Правило**: **Никогда** не вызывайте `.sync` на той же очереди, в которой уже находитесь.

#### Сценарий 3 — Deadlock с несколькими акторами (циклическая зависимость)

```swift
actor UserService {
    let authService: AuthService
    
    func getCurrentUser() async -> User? {
        await authService.validateToken()   // ← ждёт AuthService
        return User()
    }
}

actor AuthService {
    let userService: UserService
    
    func validateToken() async {
        _ = await userService.getCurrentUser()  // ← ждёт UserService
    }
}
```

**Deadlock**: циклическая зависимость между акторами.

### 3. Все основные виды Deadlock в Swift 2026

| Вид Deadlock                          | Причина                                      | Как проявляется                     | Частота в iOS-приложениях |
|---------------------------------------|----------------------------------------------|--------------------------------------|----------------------------|
| **Lock ordering deadlock**            | Захват нескольких lock в разном порядке      | Зависание потоков                    | ★★★★★                      |
| **Main queue sync deadlock**          | sync на текущей очереди (чаще main)          | Полное зависание UI                  | ★★★★☆                      |
| **Actor cycle deadlock**              | Циклическая зависимость между акторами       | Зависание асинхронных задач          | ★★★★☆ (в Swift 6+)         |
| **Semaphore / barrier deadlock**      | Неправильное использование семафоров         | Зависание задач                      | ★★★☆☆                      |
| **DispatchGroup wait deadlock**       | wait на группе внутри той же очереди         | Зависание очереди                    | ★★★☆☆                      |

### 4. Все способы избежать Deadlock в 2026

| Способ                                    | Эффективность против deadlock | Сложность | Скорость | Когда использовать в 2026    |
| ----------------------------------------- | ----------------------------- | --------- | -------- | ---------------------------- |
| **[[actor]]**                             | ★★★★★                         | ★★★☆☆     | ★★★★☆    | Основной выбор для состояния |
| **[[@MainActor]]**                        | ★★★★★                         | ★★☆☆☆     | ★★★★☆    | UI, ViewModel, SwiftUI       |
| **Фиксированный порядок lock**            | ★★★★☆                         | ★★★☆☆     | ★★★★★    | Legacy-код с [[NSLock]]      |
| **[[DispatchQueue]] [[serial]]**          | ★★★★☆                         | ★★☆☆☆     | ★★★★☆    | Простые случаи               |
| **os_unfair_lock + try lock**             | ★★★★☆                         | ★★★★☆     | ★★★★★    | Высокопроизводительные места |
| **Avoid sync on current queue**           | ★★★★★                         | ★★☆☆☆     | ★★★★★    | Главный поток                |
| **Strict Concurrency Checking (Swift 6)** | ★★★★★                         | ★★★★☆     | ★★★★★    | Обязательно в новых проектах |

### 5. Самые безопасные конструкции 2026

#### actor — золотой стандарт

```swift
actor SafeCache {
    private var storage: [String: Data] = [:]
    
    func store(_ data: Data, key: String) {
        storage[key] = data
    }
    
    func fetch(key: String) -> Data? {
        storage[key]
    }
}

let cache = SafeCache()

Task.detached {
    await cache.store(Data(), key: "profile")
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

#### Фиксированный порядок lock (если всё ещё используете NSLock)

```swift
let lockA = NSLock()
let lockB = NSLock()

// Всегда один и тот же порядок!
func safeWork() {
    lockA.lock()
    lockB.lock()
    // работа
    lockB.unlock()
    lockA.unlock()
}
```

### 6. Как найти Deadlock в 2026

| Инструмент / Способ                     | Что ловит                        | Где включается                   | Эффективность |
| --------------------------------------- | -------------------------------- | -------------------------------- | ------------- |
| **Thread Sanitizer**                    | [[data race]], Deadlock (иногда) | [[Xcode]] → Scheme → Diagnostics | ★★★★☆         |
| **Instruments → Thread Analyzer**       | Deadlock, waiting threads        | Instruments                      | ★★★★★         |
| **Xcode Debugger** (pause + threads)    | Зависшие потоки                  | Debug navigator → Threads        | ★★★★☆         |
| **lldb** команда `thread backtrace all` | Все стеки потоков                | В отладчике                      | ★★★★☆         |
| **os_signpost** + Instruments           | Долгие блокировки                | Instruments → Points of Interest | ★★★☆☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит многие потенциальные deadlock на этапе компиляции  
- **actor** — основной способ хранения изменяемого состояния  
- **@MainActor** — всё, что касается UI и SwiftUI  
- **Sendable** — всё, что передаётся между акторами  
- **Никогда** не используйте `.sync` на текущей очереди (особенно main)  
- **Фиксированный порядок захвата lock** — если используете NSLock/os_unfair_lock  
- **Избегайте вложенных sync** на одной очереди  
- **Для тестов** — используйте [[actor]] + [[XCTest]] с [[await]]  
- **Для legacy-кода** — постепенно оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Deadlock — это когда два потока стоят и ждут, пока другой первым пропустит.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> NSLock и sync — это уже legacy. Забудьте про них в новом коде.»
