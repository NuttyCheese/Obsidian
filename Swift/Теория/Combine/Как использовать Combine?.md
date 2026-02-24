Combine — это **фреймворк реактивного программирования** от Apple, встроенный в [[iOS]] 13+ / macOS 10.15+.  
Он позволяет работать с **асинхронными потоками данных** (изменения UI, сеть, уведомления, таймеры и т.д.) в декларативном стиле.

### Основные сущности Combine (коротко и чётко)

| Компонент              | Что это                                                                      | Аналогия из RxSwift / других |
| ---------------------- | ---------------------------------------------------------------------------- | ---------------------------- |
| **[[Publisher]]**      | Источник данных (может испускать значения, ошибки, завершаться)              | Observable / Publisher       |
| **[[Subscriber]]**     | Получатель данных (sink, assign, receive)                                    | Observer / Subscriber        |
| **[[Subscription]]**   | Связь между [[Publisher]] и [[Subscriber]] (управляет жизненным циклом)      | Disposable / Subscription    |
| **[[Operator]]**       | Преобразователь потока ([[map]], [[filter]], combineLatest, debounce и т.д.) | Operator                     |
| **[[AnyCancellable]]** | Токен подписки — отменяет подписку при deinit                                | DisposeBag (но без мешка)    |

### Шаг 1. Импорт и базовый синтаксис

```swift
import Combine
```

Самый простой Publisher → Subscriber:

```swift
let publisher = Just("Hello, Combine!")  // Publisher, который испускает одно значение и завершается

publisher
    .sink { value in
        print("Получено:", value)         // → "Получено: Hello, Combine!"
    }
    .store(in: &cancellables)             // ← обязательно храним подписку
```

### Шаг 2. Хранение подписок (самое важное правило 2026)

Всегда храните `AnyCancellable` в коллекции:

```swift
private var cancellables = Set<AnyCancellable>()
```

Или массив (редко):

```swift
private var cancellables: [AnyCancellable] = []
```

**Почему это обязательно?**  
Без хранения подписки не отменяются → [[retain cycle]] → утечки памяти.

### Шаг 3. Самые популярные источники Publisher в UIKit-приложении

#### 1. Изменения в [[UITextField]] / [[UITextView]]

```swift
textField.publisher(for: \.text)
    .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
    .compactMap { $0 }
    .sink { [weak self] text in
        self?.validate(text)
    }
    .store(in: &cancellables)
```

#### 2. @Published в ViewModel ([[MVVM (Model-View-ViewModel) Architecture|MVVM]] + [[Combine]])

```swift
class ProfileViewModel: ObservableObject {
    @Published var username = ""
    @Published var isValid = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $username
            .map { $0.count >= 3 }
            .assign(to: &$isValid)  // ← удобный синтаксис с iOS 14+
    }
}
```

В контроллере:

```swift
viewModel.$isValid
    .receive(on: DispatchQueue.main)
    .sink { [weak self] isValid in
        self?.updateUI(isValid: isValid)
    }
    .store(in: &cancellables)
```

#### 3. Сетевые запросы ([[URLSession]] + [[Combine]])

```swift
struct Post: Codable {
    let id: Int
    let title: String
}

URLSession.shared.dataTaskPublisher(for: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
    .map(\.data)
    .decode(type: [Post].self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { completion in
        if case .failure(let error) = completion {
            print("Ошибка:", error)
        }
    }, receiveValue: { posts in
        print("Получено постов:", posts.count)
    })
    .store(in: &cancellables)
```

#### 4. [[NotificationCenter]]

```swift
NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification)
    .sink { _ in
        print("Приложение снова на экране")
    }
    .store(in: &cancellables)
```

### Шаг 4. Самые полезные операторы Combine (топ-15 в 2026)

| Оператор                 | Что делает                                                  | Самый частый кейс           |
| ------------------------ | ----------------------------------------------------------- | --------------------------- |
| [[map]]                  | Преобразует каждое значение                                 | Изменение типа данных       |
| [[compactMap]]           | map + фильтрация [[nil]]                                    | Извлечение optional         |
| [[filter]]               | Пропускает только подходящие значения                       | Отсечение пустых строк      |
| `debounce`               | Ждёт паузу перед испусканием значения                       | Поиск с задержкой           |
| `receive(on:)`           | Переключает выполнение на указанный scheduler               | Обновление UI на main       |
| `assign(to:)`            | Присваивает значение @Published свойству                    | Связь ViewModel → View      |
| [[sink]]                 | Основной способ подписки (receiveValue + receiveCompletion) | Всё остальное               |
| `eraseToAnyPublisher`    | Скрывает конкретный тип Publisher                           | Возврат из функции          |
| `combineLatest`          | Комбинирует последние значения нескольких publishers        | Формы (email + password)    |
| `merge` / `zip`          | Объединяет потоки                                           | Несколько источников данных |
| `catch` / `replaceError` | Обрабатывает ошибки                                         | Отображение fallback        |
| `retry`                  | Повторяет подписку при ошибке                               | Сетевые запросы             |
| `share`                  | Делает подписку общей (один запрос — много подписчиков)     | Дорогие операции            |
| `multicast`              | Более гибкая версия share                                   | Редко                       |
| `handleEvents`           | Логирование событий (receiveSubscription и т.д.)            | Отладка                     |

### Шаг 5. Полный пример ViewModel + [[UIKit]] (MVVM + Combine)

```swift
@MainActor
class UserProfileViewModel: ObservableObject {
    
    @Published var username = ""
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var user: User?
    
    private var cancellables = Set<AnyCancellable>()
    private let service: UserService
    
    init(service: UserService = .live) {
        self.service = service
        
        // Автоматическая валидация
        $username
            .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
            .map { $0.count >= 3 }
            .assign(to: &$isValid)
        
        // Автоматическая загрузка при изменении username
        $username
            .filter { $0.count >= 3 }
            .debounce(for: .seconds(0.8), scheduler: DispatchQueue.main)
            .sink { [weak self] name in
                self?.fetchUser(name: name)
            }
            .store(in: &cancellables)
    }
    
    private func fetchUser(name: String) {
        isLoading = true
        errorMessage = nil
        
        service.fetchUser(by: name)
            .receive(on: DispatchQueue.main)
            .sink { [weak self] completion in
                self?.isLoading = false
                if case .failure(let error) = completion {
                    self?.errorMessage = error.localizedDescription
                }
            } receiveValue: { [weak self] user in
                self?.user = user
            }
            .store(in: &cancellables)
    }
}
```

### Итог: как начать использовать Combine в 2026 году

1. Импортируй `import Combine`
2. Создай `@Published` свойства в ViewModel
3. Подписывайся через `.sink` и сохраняй в `Set<AnyCancellable>`
4. Используй операторы для трансформации (map, filter, debounce, receive(on:))
5. Для сетевых запросов — `URLSession.dataTaskPublisher`
6. Для уведомлений — `NotificationCenter.default.publisher(for:)`
7. Для UI — `.receive(on: DispatchQueue.main)`

Combine — мощный инструмент, но в 2026 году часто комбинируют его с **Swift Concurrency** ([[async]]/[[await]] + [[Task]]):

```swift
Task {
    do {
        let user = try await service.fetchUser(by: username)
        await MainActor.run { self.user = user }
    } catch {
        await MainActor.run { self.errorMessage = error.localizedDescription }
    }
}
```

**Вывод**:  
Combine идеален для реактивного UI, форм, сетевых цепочек и MVVM.  
Но если проект новый и простой — можно обойтись **async/await + @Observable**.  
Если же нужно мощное комбинирование потоков — Combine всё ещё король.
