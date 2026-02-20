**Priority Inversion** — это ситуация, когда **высокоприоритетный поток** (например, задача с [[QoS]] `.userInitiated` или `.userInteractive`) **не может быстро выполниться**, потому что он ждёт ресурс, который удерживает **низкоприоритетный поток** (QoS `.background` или `.utility`).

В результате **важная задача** (анимация, реакция на касание, обработка аудио) **задерживается** из-за **неважной задачи** (фоновый анализ, индексация, загрузка контента).

**Классическая аналогия**:
- Важный гость (high-priority) хочет войти в комнату.
- Дверь заперта.
- Ключ у уборщика (low-priority), который медленно моет пол.
- Уборщик не торопится, потому что его приоритет низкий.
- Гость стоит и ждёт → инверсия приоритетов.

### 2. Как возникает Priority Inversion (классический сценарий)

```swift
let lock = NSLock()

// Низкоприоритетный поток (фоновая задача)
DispatchQueue.global(qos: .background).async {
    lock.lock()
    print("Low-priority захватил lock")
    
    // Долгая работа (например, обработка фото, индексация)
    Thread.sleep(forTimeInterval: 5)
    
    lock.unlock()
}

// Высокоприоритетный поток (UI-реакция)
DispatchQueue.global(qos: .userInitiated).async {
    print("High-priority пытается захватить lock...")
    lock.lock()           // ← ждёт 5 секунд!
    print("High-priority захватил lock")
    lock.unlock()
}
```

**Что происходит**:
1. Low-priority поток захватывает lock и начинает долгую работу.
2. High-priority поток приходит и пытается захватить lock → блокируется.
3. Планировщик ОС видит, что high-priority поток заблокирован → **временно повышает приоритет** low-priority потока (это называется **priority inheritance**), чтобы он быстрее освободил lock.
4. Но если есть **среднеприоритетный поток** (например, `.default`), он может вклиниться и оттягивать время → high-priority всё равно ждёт дольше.

**Результат**: задержка UI, анимации «дергаются», приложение кажется «залипшим».

### 3. Самые частые сценарии Priority Inversion в iOS-приложениях 2026

| №   | Сценарий                                                | Как проявляется                              | Частота |
| --- | ------------------------------------------------------- | -------------------------------------------- | ------- |
| 1   | Долгая работа под `NSLock` / `os_unfair_lock` в фоне    | UI зависает при касании / анимации           | ★★★★★   |
| 2   | [[Core Data]] / [[Swift/Теория/Сторонние библиотеки и расширения/Realm]] save в background + UI update | Таблица не обновляется вовремя               | ★★★★☆   |
| 3   | Фоновая индексация / аналитика держит lock              | Реакция на касание задерживается             | ★★★★☆   |
| 4   | Несколько акторов с циклической зависимостью            | Асинхронные задачи зависают                  | ★★★★☆   |
| 5   | DispatchQueue.concurrent + sync/wait                    | Задержка в обработке пользовательского ввода | ★★★☆☆   |

### 4. Все современные способы защиты от Priority Inversion (2026)

| Способ                                                  | Защита от Priority Inversion | Скорость | Сложность | Рекомендация 2026            | Примечание                    |
| ------------------------------------------------------- | ---------------------------- | -------- | --------- | ---------------------------- | ----------------------------- |
| **actor**                                               | ★★★★★                        | ★★★★☆    | ★★★☆☆     | Основной выбор               | Apple рекомендует             |
| **@MainActor**                                          | ★★★★★                        | ★★★★☆    | ★★☆☆☆     | UI и SwiftUI                 | Стандарт для ViewModel        |
| **os_unfair_lock** + **минимальная критическая секция** | ★★★★☆                        | ★★★★★    | ★★★★☆     | Высокопроизводительные места | Только если actor не подходит |
| **[[DispatchQueue]] [[serial]]**                        | ★★★★☆                        | ★★★★☆    | ★★☆☆☆     | Legacy-код                   | Всё ещё актуально             |
| **Swift 6 strict concurrency + Sendable**               | ★★★★★                        | ★★★★★    | ★★★★☆     | Новые проекты                | Обязательно                   |
| **Avoid long-running work under lock**                  | ★★★★★                        | ★★★★★    | ★★☆☆☆     | Всегда                       | Самое важное правило          |
| **Priority inheritance** (системный)                    | ★★★★☆                        | —        | —         | Автоматически                | [[iOS]]/macOS помогает        |

### 5. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### actor — золотой стандарт (нет Priority Inversion при корректном использовании)

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]
    
    func store(_ image: UIImage, for url: URL) {
        cache[url] = image
    }
    
    func fetch(for url: URL) -> UIImage? {
        cache[url]
    }
}

let cache = ImageCache()

Task(priority: .userInitiated) {
    let image = await downloadImage()
    await cache.store(image, for: url)
}
```

#### @MainActor для UI и ViewModel

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    
    func load() async {
        let fetched = try await fetchUser()
        user = fetched  // безопасно и без инверсии
    }
}
```

#### os_unfair_lock + минимальная критическая секция (если actor не подходит)

```swift
let lock = os_unfair_lock()
var cache: [String: Data] = [:]

func getData(key: String) -> Data? {
    os_unfair_lock_lock(&lock)
    defer { os_unfair_lock_unlock(&lock) }
    
    return cache[key]  // только чтение — очень быстро
}
```

### 6. Как найти Priority Inversion в 2026

| Инструмент / Способ               | Что ловит                                        | Где включается                   | Эффективность |
| --------------------------------- | ------------------------------------------------ | -------------------------------- | ------------- |
| **Instruments → Thread Analyzer** | Долгие блокировки, инверсия приоритетов          | Instruments                      | ★★★★★         |
| **os_signpost + Instruments**     | Длительность критических секций                  | Instruments → Points of Interest | ★★★★☆         |
| **Xcode Debugger + Threads**      | Зависшие high-priority потоки                    | Debug navigator → Threads        | ★★★★☆         |
| **Thread Sanitizer**              | [[data race]] (иногда помогает увидеть контекст) | [[Xcode]] → Diagnostics          | ★★★★☆         |
| **Signposts + custom logging**    | Время удержания lock / actor                     | Console / os_signpost            | ★★★☆☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking + actor помогают избежать большинства проблем  
- **actor** — основной способ хранения изменяемого состояния  
- **@MainActor** — всё, что касается UI и SwiftUI  
- **Sendable** — всё, что передаётся между акторами  
- **Минимизируйте критическую секцию** — захватывайте lock только на время изменения данных  
- **Не держите lock на сетевых запросах, I/O, тяжёлых вычислениях**  
- **Используйте [[defer]] `{ unlock() }`** — спасает от забытого unlock  
- **Для тестов** — используйте [[actor]] + [[XCTest]] с [[await]]  
- **Для legacy-кода** — постепенно оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Priority Inversion — это когда важный гость ждёт, пока уборщик медленно моет пол.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> Длинные lock, sync, глобальные var — это вчерашний день. Забудьте про них в новом коде.»
