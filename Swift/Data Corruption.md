**Data Corruption** — это ситуация, когда данные в памяти или на диске становятся **неконсистентными**, **непредсказуемыми** или **повреждёнными** из-за некорректного параллельного доступа.

Самые частые причины в [[iOS]]/[[Swift]]:

- **[[race condition]]** (гонка данных) — два или более потока одновременно читают/пишут в одну и ту же память  
- **Неправильная синхронизация** (или её отсутствие)  
- **Неправильное использование [[Value Type]]** в многопоточной среде  
- **Передача mutable объектов между акторами без изоляции**  
- **Ошибки с Unsafe** (UnsafeMutablePointer, UnsafeRawPointer и т.д.)

### 2. Симптомы Data Corruption (что вы увидите в приложении)

| Симптом                                        | Как проявляется в iOS-приложении                             | Частота в 2026 |
| ---------------------------------------------- | ------------------------------------------------------------ | -------------- |
| Неправильные / случайные значения              | Счётчик не доходит до нужного числа, массив содержит мусор   | ★★★★★          |
| Краш с EXC_BAD_ACCESS / SIGSEGV                | Обращение к освобождённой памяти или [[nil]]                 | ★★★★☆          |
| Неконсистентное состояние UI                   | Кнопка активна, но данные не загрузились                     | ★★★★☆          |
| Повреждение базы данных / [[Core Data]]        | Дубликаты записей, потеря связей                             | ★★★☆☆          |
| Пропадание аналитики / событий                 | Некоторые события не доходят до сервера                      | ★★★☆☆          |
| Случайные краши только на реальных устройствах | Thread Sanitizer в симуляторе не ловит, а на устройстве — да | ★★★★☆          |

### 3. Классические примеры Data Corruption (и как они выглядят в 2026)

#### Пример 1 — Гонка при записи в [[Int]] (самая частая ошибка)

```swift
var counter = 0

Task.detached {  // или DispatchQueue.global().async
    for _ in 0..<1_000_000 {
        counter += 1
    }
}

Task.detached {
    for _ in 0..<1_000_000 {
        counter += 1
    }
}

// Результат: counter почти никогда не будет ровно 2_000_000
```

**Почему ломается**: операция `+=` не атомарна (чтение → сложение → запись).

#### Пример 2 — Гонка в массиве (очень частая в UI)

```swift
var images: [UIImage] = []

func downloadImage(url: URL) async {
    let (data, _) = try! await URLSession.shared.data(from: url)
    let image = UIImage(data: data)!
    
    images.append(image)  // ← гонка!
}
```

**Результат**: массив может содержать nil, повреждённые объекты или крашиться.

#### Пример 3 — Ошибка с @Published / Observable в [[SwiftUI]]

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func load() {
        Task.detached {  // ← ошибка!
            let newItems = try await fetchItems()
            items += newItems  // ← гонка, даже с @MainActor
        }
    }
}
```

**В Swift 6**: компилятор может выдать предупреждение, но в Swift 5.10+ это всё ещё может молча сломать данные.

### 4. Все современные способы защиты от Data Corruption (2026)

| Способ защиты                         | Уровень безопасности | Скорость | Сложность | Когда использовать в 2026             |
| ------------------------------------- | -------------------- | -------- | --------- | ------------------------------------- |
| **[[actor]]**                         | ★★★★★                | ★★★★☆    | ★★★☆☆     | Основной выбор для состояния          |
| **[[@MainActor]] / [[@globalActor]]** | ★★★★★                | ★★★★☆    | ★★☆☆☆     | UI, ViewModel, глобальные сервисы     |
| **[[Sendable]]** + strict concurrency | ★★★★★                | ★★★★★    | ★★★★☆     | Передача данных между акторами        |
| **[[DispatchQueue]]** ([[serial]])    | ★★★★☆                | ★★★★☆    | ★★☆☆☆     | Legacy-код, простые случаи            |
| **[[NSLock]] / os_unfair_lock**       | ★★★★☆                | ★★★★★    | ★★★☆☆     | Высокопроизводительные примитивы      |
| **Atomic** (swift-atomics)            | ★★★★☆                | ★★★★★    | ★★★☆☆     | Простые примитивы ([[Int]], [[Bool]]) |
| **@Atomic / ManagedAtomic**           | ★★★★☆                | ★★★★★    | ★★★☆☆     | Низкоуровневые оптимизации            |
| **Value types + immutability**        | ★★★★★                | ★★★★★    | ★★☆☆☆     | Максимальная безопасность             |

### 5. Полные безопасные примеры (2026 стандарт)

#### Пример 1 — actor (самый рекомендуемый способ)

```swift
actor Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        value
    }
}

let counter = Counter()

Task.detached {
    for _ in 0..<100_000 {
        await counter.increment()
    }
}

Task.detached {
    for _ in 0..<100_000 {
        await counter.increment()
    }
}

// 100% безопасно, всегда будет 200_000
```

#### Пример 2 — @MainActor для UI

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func load() async {
        let newItems = try await fetchItems()
        items += newItems  // безопасно — @MainActor
    }
}
```

#### Пример 3 — [[Sendable]] и передача данных между акторами

```swift
actor DataStore {
    private var cache: [String: Data] = [:]
    
    func store(_ data: Data, for key: String) {
        cache[key] = data
    }
}

struct ImageLoader: Sendable {
    let store: DataStore
    
    func load(url: URL) async throws {
        let (data, _) = try await URLSession.shared.data(from: url)
        await store.store(data, for: url.absoluteString)
    }
}
```

### 6. Как найти Data Corruption в 2026

| Инструмент / Способ                     | Что ловит                              | Где включается                   | Эффективность |
| --------------------------------------- | -------------------------------------- | -------------------------------- | ------------- |
| **Thread Sanitizer**                    | [[race condition]], [[data race]]     | [[Xcode]] → Scheme → Diagnostics | ★★★★★         |
| **Swift 6 Strict Concurrency Checking** | Передача mutable данных между акторами | Build Settings → Swift Compiler  | ★★★★★         |
| **Address Sanitizer**                   | Use-after-free, buffer overflow        | Xcode → Diagnostics              | ★★★★☆         |
| **Instruments → Thread Sanitizer**      | Race conditions в [[Runtime]]          | Instruments                      | ★★★★☆         |
| **Xcode Previews + @MainActor**         | UI-гонки                               | SwiftUI Previews                 | ★★★☆☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит большинство гонок на этапе компиляции  
- **actor** — основной способ хранения изменяемого состояния  
- **@MainActor** — всё, что связано с UI и SwiftUI  
- **Sendable** — всё, что передаётся между акторами (структуры, [[final]]-классы)  
- **Value types + immutability** — используйте [[let]], [[struct]], копирование  
- **Избегайте глобальных var** — даже с синхронизацией  
- **Для коллекций** — используйте `actor`, `CurrentValueSubject`, `@Published` с @MainActor  
- **Для тестов** — используйте `actor` + `XCTest` с `await`  
- **Для legacy-кода** — оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Data Corruption — это когда несколько рук тянутся к одной памяти одновременно.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> Всё остальное — это костыли или legacy.»
