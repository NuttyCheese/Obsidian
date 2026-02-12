### 1. Что такое Data Race (гонка данных)

**Data Race** — это ситуация, когда **два или более потока одновременно обращаются к одной и той же области памяти**, и хотя бы один из них **пишет** в неё, **без какой-либо синхронизации**.

Согласно стандарту C++11 / C11 (и аналогично в [[Swift]]):

> Data Race возникает, если хотя бы один поток выполняет **запись**, а другой — **чтение или запись** в ту же память без упорядочивания (synchronizes-with).

**Последствия Data Race** (по убыванию частоты в [[iOS]]-приложениях 2026):

1. Неправильные / случайные значения переменных  
2. Краш с `EXC_BAD_ACCESS` / `SIGSEGV` / `SIGBUS`  
3. Повреждение кучи / стека (heap corruption)  
4. Неконсистентное состояние UI  
5. Повреждение базы данных / [[Core Data]] / [[Realm]]  
6. Утечки памяти / use-after-free  
7. Случайные краши только на реальных устройствах (Thread Sanitizer не всегда ловит в симуляторе)

### 2. Самые частые примеры Data Race в Swift 2026

#### Пример 1 — Гонка при записи в Int (самая частая ошибка)

```swift
var counter = 0

Task.detached {
    for _ in 0..<1_000_000 {
        counter += 1  // ← гонка!
    }
}

Task.detached {
    for _ in 0..<1_000_000 {
        counter += 1  // ← гонка!
    }
}

// Результат: counter почти никогда не будет ровно 2_000_000
// Возможные значения: от ~1_000_000 до ~1_999_999
```

**Почему ломается**: операция `+=` состоит из трёх шагов (чтение → сложение → запись), которые не атомарны.

#### Пример 2 — Гонка в массиве (очень частая в UI)

```swift
var downloadedImages: [UIImage] = []

Task.detached {
    for url in urls1 {
        let image = try! await downloadImage(url)
        downloadedImages.append(image)  // ← гонка!
    }
}

Task.detached {
    for url in urls2 {
        let image = try! await downloadImage(url)
        downloadedImages.append(image)  // ← гонка!
    }
}
```

**Последствия**: массив может содержать nil, повреждённые объекты, краш при доступе, дубликаты, пропуски.

#### Пример 3 — Гонка в @Published / ObservableObject ([[SwiftUI]])

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

**В Swift 6**: компилятор выдаст предупреждение или ошибку (strict concurrency checking).

### 3. Все современные способы защиты от Data Race (2026)

| Способ защиты                     | Уровень безопасности | Скорость | Сложность | Когда использовать в 2026 | Примечание |
|-----------------------------------|------------------------|----------|-----------|----------------------------|------------|
| **actor**                         | ★★★★★                  | ★★★★☆    | ★★★☆☆     | Основной выбор для изменяемого состояния | Рекомендуется Apple |
| **@MainActor**                    | ★★★★★                  | ★★★★☆    | ★★☆☆☆     | UI, ViewModel, глобальные сервисы | Стандарт для SwiftUI |
| **Sendable + strict concurrency** | ★★★★★                  | ★★★★★    | ★★★★☆     | Передача данных между акторами | Обязательно в Swift 6 |
| **@globalActor**                  | ★★★★★                  | ★★★★☆    | ★★★☆☆     | Глобальные синглтоны (например, Database) | Редко, но мощно |
| **DispatchQueue (serial)**        | ★★★★☆                  | ★★★★☆    | ★★☆☆☆     | Legacy-код, простые случаи | Всё ещё актуально |
| **os_unfair_lock**                | ★★★★☆                  | ★★★★★    | ★★★★☆     | Высокопроизводительные примитивы | Лучше NSLock |
| **Atomic** (swift-atomics)        | ★★★★☆                  | ★★★★★    | ★★★☆☆     | Простые примитивы (Int, Bool) | Для низкоуровневых оптимизаций |
| **Value types + immutability**    | ★★★★★                  | ★★★★★    | ★★☆☆☆     | Максимальная безопасность | Идеальный вариант |

### 4. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### actor — золотой стандарт

```swift
actor SafeCounter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func get() -> Int {
        value
    }
}

let counter = SafeCounter()

Task.detached {
    for _ in 0..<1_000_000 {
        await counter.increment()  // 100% безопасно
    }
}
```

#### @MainActor для UI и ViewModel

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

#### Sendable и передача данных между акторами

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

### 5. Как найти Data Race в 2026

| Инструмент / Способ                     | Что ловит                              | Где включается                  | Эффективность |
| --------------------------------------- | -------------------------------------- | ------------------------------- | ------------- |
| **Thread Sanitizer**                    | Data Races, race conditions            | Xcode → Scheme → Diagnostics    | ★★★★★         |
| **Swift 6 Strict Concurrency Checking** | Передача mutable данных между акторами | Build Settings → Swift Compiler | ★★★★★         |
| **Address Sanitizer**                   | Use-after-free, buffer overflow        | Xcode → Diagnostics             | ★★★★☆         |
| **Instruments → Thread Sanitizer**      | Data Races в [[runtime]]               | Instruments                     | ★★★★☆         |
| **Xcode Previews + @MainActor**         | UI-гонки                               | SwiftUI Previews                | ★★★☆☆         |

### 6. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит большинство гонок на этапе компиляции  
- **[[actor]]** — основной способ хранения изменяемого состояния  
- **[[@MainActor]]** — всё, что связано с UI и [[SwiftUI]]  
- **[[Sendable]]** — всё, что передаётся между акторами (структуры, [[final]]-классы)  
- **[[Value type]] + immutability** — используйте [[let]], [[struct]], копирование  
- **Избегайте глобальных [[var]]** — даже с синхронизацией  
- **Для коллекций** — используйте `actor`, `CurrentValueSubject`, `@Published` с @MainActor  
- **Для тестов** — используйте [[actor]] + [[XCTest]] с [[await]]  
- **Для legacy-кода** — оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Data Race — это когда несколько рук тянутся к одной памяти одновременно.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> Всё остальное — это костыли или legacy.»
