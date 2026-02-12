### 1. Что такое Race Condition простыми словами

**Race Condition** — это **логическая ошибка**, когда **результат программы зависит от того, в каком порядке потоки (или асинхронные задачи) выполняют свои операции**.

Если порядок выполнения меняется (а он меняется почти всегда при многопоточности) — результат становится **непредсказуемым**.

**Коротко и по-человечески**:
- Два или больше потоков/задач работают с **общими данными**.
- Они выполняют операции **не атомарно** (не за один неделимый шаг).
- Кто успел первым — тот «выиграл», а кто вторым — может получить мусор или сломать данные.

**Самые частые проявления в [[iOS]]-приложении**:

- Счётчик досчитал до 1500 вместо 2000  
- В массиве пропали/появились лишние элементы  
- Пользователь видит старые данные, хотя они уже обновились  
- Корзина в приложении показывает неправильное количество товаров  
- Аватар загрузился, но имя пользователя — нет (или наоборот)  
- Редкие, трудно воспроизводимые баги «иногда работает, иногда нет»

### 2. Race Condition vs Data Race — важное различие (2026)

| Понятие            | Что это такое                                    | Повреждает ли память?       | Ловит ли Thread Sanitizer? | Пример в Swift                  |
| ------------------ | ------------------------------------------------ | --------------------------- | -------------------------- | ------------------------------- |
| **[[Data Race]]**  | Несинхронизированное чтение/запись в одну память | Да (может повредить данные) | Да (очень надёжно)         | `counter += 1` из двух потоков  |
| **Race Condition** | Логическая зависимость от порядка выполнения     | Не обязательно              | Нет (не всегда)            | Порядок операций в двух задачах |

**Главный вывод 2026 года**:
- **Data Race** — это частный случай Race Condition, который **повреждает память** и **ловится Thread Sanitizer**.
- **Race Condition** — более широкое понятие: даже если память не повреждена, **логика всё равно сломана**.

### 3. Самые частые примеры Race Condition в Swift 2026

#### Пример 1 — Гонка при обновлении счётчика (самая классическая)

```swift
var totalPrice = 0.0

// Пользователь добавляет 1000 товаров в корзину из разных потоков
for product in products {
    Task.detached {
        let price = await fetchPrice(for: product)
        totalPrice += price  // ← Race Condition!
    }
}

// totalPrice почти никогда не будет равен сумме всех цен
```

#### Пример 2 — Гонка в массиве (очень частая в UI)

```swift
var recentSearches: [String] = []

func saveSearch(_ query: String) {
    Task.detached {
        // Имитация задержки (сеть, диск)
        try? await Task.sleep(nanoseconds: 100_000_000)
        recentSearches.append(query)  // ← Race Condition!
    }
}
```

**Возможные результаты**: дубли, пропуски, неправильный порядок, краш при одновременном append.

#### Пример 3 — Гонка в @Published / ObservableObject ([[SwiftUI]])

```swift
@MainActor
class SearchViewModel: ObservableObject {
    @Published var results: [SearchResult] = []
    
    func search(query: String) {
        Task.detached {
            let fetched = try await api.search(query)
            results += fetched  // ← Race Condition, даже с @MainActor!
        }
    }
}
```

**В Swift 6**: компилятор может выдать предупреждение, но в Swift 5.10+ это всё ещё молча ломается.

### 4. Все современные способы защиты от Race Condition (2026)

| Способ защиты                     | Защита от Race Condition | Скорость | Сложность | Рекомендация 2026 | Примечание |
|-----------------------------------|---------------------------|----------|-----------|-------------------|------------|
| **actor**                         | ★★★★★                     | ★★★★☆    | ★★★☆☆     | Основной выбор    | Apple рекомендует |
| **@MainActor**                    | ★★★★★                     | ★★★★☆    | ★★☆☆☆     | UI и ViewModel    | Стандарт для SwiftUI |
| **Sendable + strict concurrency** | ★★★★★                     | ★★★★★    | ★★★★☆     | Новые проекты     | Обязательно в Swift 6 |
| **DispatchQueue (serial)**        | ★★★★☆                     | ★★★★☆    | ★★☆☆☆     | Legacy-код        | Всё ещё актуально |
| **os_unfair_lock**                | ★★★★☆                     | ★★★★★    | ★★★★☆     | Высокопроизводительные места | Для примитивов |
| **Atomic** (swift-atomics)        | ★★★★☆                     | ★★★★★    | ★★★☆☆     | Простые счётчики  | Для Int, Bool и т.д. |
| **Value types + immutability**    | ★★★★★                     | ★★★★★    | ★★☆☆☆     | Идеальный вариант | Предпочтительно всегда |

### 5. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### [[actor]] — золотой стандарт

```swift
actor ShoppingCart {
    private var items: [Product] = []
    private var totalPrice: Decimal = 0
    
    func add(_ product: Product) {
        items.append(product)
        totalPrice += product.price
    }
    
    func getTotal() -> Decimal {
        totalPrice
    }
}

let cart = ShoppingCart()

Task.detached {
    await cart.add(product1)
    await cart.add(product2)
}
```

#### [[@MainActor]] для UI и ViewModel

```swift
@MainActor
class CartViewModel: ObservableObject {
    @Published var items: [Product] = []
    @Published var total: Decimal = 0
    
    func add(_ product: Product) async {
        items.append(product)
        total += product.price  // безопасно
    }
}
```

#### [[Sendable]] и передача данных между акторами

```swift
actor Analytics {
    private var events: [Event] = []
    
    func log(_ event: Event) {
        events.append(event)
    }
}

struct Event: Sendable {
    let name: String
    let timestamp: Date
}

func trackEvent(_ name: String) {
    Task.detached {
        let event = Event(name: name, timestamp: Date())
        await analytics.log(event)  // безопасно
    }
}
```

### 6. Как найти Race Condition в 2026

| Инструмент / Способ                     | Что ловит                              | Где включается                   | Эффективность |
| --------------------------------------- | -------------------------------------- | -------------------------------- | ------------- |
| **Thread Sanitizer**                    | Data Race (основной инструмент)        | [[Xcode]] → Scheme → Diagnostics | ★★★★★         |
| **Swift 6 Strict Concurrency Checking** | Передача mutable данных между акторами | Build Settings → Swift Compiler  | ★★★★★         |
| **Address Sanitizer**                   | Use-after-free, buffer overflow        | Xcode → Diagnostics              | ★★★★☆         |
| **Instruments → Thread Sanitizer**      | Race Condition в runtime               | Instruments                      | ★★★★☆         |
| **Xcode Runtime Issues**                | Нарушения в runtime                    | Debug navigator → Runtime Issues | ★★★★☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит большинство Race Condition на этапе компиляции  
- **[[actor]]** — основной способ хранения изменяемого состояния  
- **[[@MainActor]]** — всё, что касается UI и [[SwiftUI]]  
- **[[Sendable]]** — всё, что передаётся между акторами (структуры, [[final]]-классы)  
- **[[Value type]] + immutability** — используйте `let`, `struct`, копирование  
- **Избегайте глобальных var** — даже с синхронизацией  
- **Для коллекций** — используйте `actor`, `CurrentValueSubject`, `@Published` с @MainActor  
- **Для тестов** — используйте `actor` + `XCTest` с `await`  
- **Для legacy-кода** — постепенно оборачивайте в `actor` или серийную очередь

**Короткий девиз 2026**:
> «Race Condition — это когда результат зависит от того, кто первый добежал до данных.  
> В 2026 году ответ один: actor + @MainActor + Sendable + strict concurrency.  
> Гонки, замки, глобальные var — это уже вчерашний день. Забудьте про них в новом коде.»
