**`Cancellable`** — это **протокол** в фреймворке **[[Combine]]**, который описывает объект, способный **отменить** выполняемую работу (обычно — подписку на [[Publisher]]).

Любой тип, реализующий `Cancellable`, обязан иметь единственный метод:

```swift
protocol Cancellable {
    func cancel()
}
```

**Самая популярная** и **единственная** часто используемая реализация — это **`AnyCancellable`**.

### Зачем нужен Cancellable и почему именно AnyCancellable

| Проблема без Cancellable                            | Как решает Cancellable / [[AnyCancellable]]                              | Почему это критично в 2026          |
| --------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------- |
| Подписка на Publisher живёт вечно                   | При `cancel()` или [[deinit]] → подписка отменяется                      | Нет утечек памяти и лишних запросов |
| Разные Publisher возвращают разные типы Cancellable | `AnyCancellable` — type-erased, можно хранить в коллекции                | Единый тип для всех подписок        |
| Забыть отменить подписку                            | `AnyCancellable` + `.store(in: &cancellables)` — почти невозможно забыть | Предотвращает [[retain cycle]]      |
| Ручное управление жизненным циклом                  | Автоматическая отмена при выходе из scope ([[deinit]])                   | Чистый и безопасный код             |

### Самые важные и актуальные способы использования (2026)

#### 1. Самый популярный паттерн — .store(in: &cancellables)

```swift
class ViewModel: ObservableObject {
    
    @Published var searchText = ""
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $searchText
            .debounce(for: .seconds(0.5), scheduler: RunLoop.main)
            .removeDuplicates()
            .sink { [weak self] text in
                self?.performSearch(text)
            }
            .store(in: &cancellables)  // ← ключевой момент
    }
    
    private func performSearch(_ query: String) {
        // поиск
    }
}
```

**Почему Set<AnyCancellable> — золотой стандарт:**
- Автоматически отменяет все подписки при deinit ViewModel
- Одна строка `.store(in: &cancellables)`
- Работает с любым типом Publisher
- Thread-safe в большинстве случаев

#### 2. Хранение одной подписки (когда подписок мало)

```swift
class SomeController {
    private var subscription: AnyCancellable?
    
    func startObserving() {
        subscription = NotificationCenter.default.publisher(for: .someEvent)
            .sink { _ in
                // обработка
            }
    }
    
    func stopObserving() {
        subscription?.cancel()
        subscription = nil
    }
}
```

#### 3. Кастомная реализация Cancellable (редко, но полезно)

```swift
class NetworkTask: Cancellable {
    
    private var task: URLSessionDataTask?
    private var onCancel: (() -> Void)?
    
    init(task: URLSessionDataTask, onCancel: @escaping () -> Void) {
        self.task = task
        self.onCancel = onCancel
    }
    
    func cancel() {
        task?.cancel()
        onCancel?()
        task = nil
        onCancel = nil
    }
}

// Использование
let task = NetworkTask(task: dataTask) {
    print("Запрос отменён")
}
task.cancel()
```

#### 4. AnyCancellable как замыкание (очень популярный трюк)

```swift
let cancellable = AnyCancellable {
    print("Подписка отменена")
    // здесь можно вызвать cleanup: invalidate timer, remove observer и т.д.
}

cancellables.insert(cancellable)
// или просто
cancellable.cancel()
```

### Лучшие практики Cancellable / AnyCancellable в Swift 2026

- **Всегда** храните подписки в `private var cancellables = Set<AnyCancellable>()`  
- **Никогда** не храните `AnyCancellable` в глобальных переменных — это утечка  
- **В SwiftUI** — подписки обычно живут в `@StateObject` / `@ObservableObject` → автоматически отменяются  
- **Для долгоживущих подписок** (глобальный EventBus, background monitoring) — храните в singleton с явной отменой  
- **Для Combine + async/await** — часто подписки заменяют на `Task` → AnyCancellable не нужен  
- **Для отладки** — используйте `handleEvents(receiveCancel: { print("Отмена!") })`  
- **Документируйте** — пишите комментарий:

```swift
private var cancellables = Set<AnyCancellable>() // все Combine-подписки контроллера
```

**Короткий итог 2026**:
> `Cancellable` — протокол, `AnyCancellable` — универсальная реализация для отмены подписки в Combine.  
> В 2026 году:  
> - стандартный паттерн — `.sink(...).store(in: &cancellables)`  
> - хранилище — `Set<AnyCancellable>` или массив  
> - предотвращает утечки памяти и ненужные операции  
> - это **обязательный** инструмент для любого кода на Combine  
