Паттерн Observer **не устарел**, но его **классическая реализация** (Subject + список Observer + add/remove/notify) используется **очень редко**.

Почему так произошло:

| Период    | Основной способ реализации Observer                                    | Актуальность в 2026 |
| --------- | ---------------------------------------------------------------------- | ------------------- |
| 2010–2019 | Классический [[GoF]] (Subject + массив observers)                      | ★★★☆☆               |
| 2020–2023 | [[NotificationCenter]] + [[Combine]] [[Publisher]]                     | ★★★★☆               |
| 2024–2026 | `@Published` / `@Observable` / `AsyncSequence` / [[TCA]] / [[SwiftUI]] | ★★★★★               |

**Ключевой вывод 2026**:
> Классический ручной Observer (как в учебниках GoF) почти не используется в новом коде.  
> Вместо него применяются **встроенные механизмы языка** и **фреймворки**, которые уже реализуют Observer под капотом.

### 2. Современные способы реализации Observer в Swift 2026

| Способ                                             | Когда использовать в 2026 году                    | Преимущества                                | Минусы                               | Примерный % использования  |
| -------------------------------------------------- | ------------------------------------------------- | ------------------------------------------- | ------------------------------------ | -------------------------- |
| `@Published` + `@ObservableObject` / `@Observable` | [[SwiftUI]], ViewModel → View обновление          | Самый простой и рекомендуемый               | Только для UI / ObservableObject     | ★★★★★ (60–70%)             |
| Combine Publisher / Subscriber                     | [[iOS]] 13+, сложные цепочки реактивности         | Мощный, гибкий, declarative                 | Крутая кривая обучения, retain cycle | ★★★★☆ (20–25%)             |
| `AsyncSequence` / `AsyncStream`                    | Потоки событий, сервер-сайд события, стримы       | Полностью [[async]]/[[await]], [[Sendable]] | Нет автосабскрайба                   | ★★★★☆ (10–15%)             |
| TCA (The Composable Architecture)                  | Приложения с unidirectional data flow             | Всё в reducer’ах, предсказуемость           | Требует изучения TCA                 | ★★★★☆ (в крупных проектах) |
| NotificationCenter                                 | Системные события, legacy, межмодульная связь     | Работает везде, нет зависимостей            | Сложно отлаживать, [[retain cycle]]  | ★★☆☆☆ (legacy)             |
| Классический GoF Observer (ручной)                 | Специфические случаи, когда нужен полный контроль | Полный контроль над жизненным циклом        | Много boilerplate, retain cycle      | ★☆☆☆☆ (очень редко)        |

### 3. Самые популярные и рекомендуемые реализации в 2026 году

#### Вариант 1 — Самый частый: `@Observable` + `@ObservationIgnored` (SwiftUI + Swift 6+)

```swift
@Observable
final class UserViewModel {
    var name = "Загрузка..."
    var avatar: UIImage?
    
    private let service: any UserFetching
    
    init(service: any UserFetching) {
        self.service = service
    }
    
    func load() async {
        do {
            let user = try await service.fetchCurrentUser()
            name = user.name
            
            if let url = user.avatarURL {
                avatar = try await loadImage(from: url)
            }
        } catch {
            name = "Ошибка"
        }
    }
}

// В SwiftUI
struct ProfileView: View {
    @State private var vm = UserViewModel(service: APIService())
    
    var body: some View {
        VStack {
            Text(vm.name)
            if let avatar = vm.avatar {
                Image(uiImage: avatar)
            }
        }
        .task { await vm.load() }
    }
}
```

**Почему это лучший выбор в 2026**:
- `@Observable` → **автоматическое** обновление UI  
- Нет `objectWillChange`, нет retain cycle  
- Полная поддержка strict concurrency  
- Минимум кода

#### Вариант 2 — Combine Publisher (всё ещё очень живой)

```swift
final class UserService: ObservableObject {
    @Published var users: [User] = []
    @Published var error: Error?
    
    private var cancellables = Set<AnyCancellable>()
    
    func load() {
        URLSession.shared.dataTaskPublisher(for: usersURL)
            .map(\.data)
            .decode(type: [User].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    if case .failure(let err) = completion {
                        self?.error = err
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}
```

**Когда всё ещё используют Combine в 2026**:
- проекты на iOS 13–15  
- сложные цепочки операторов  
- интеграция с legacy-кодом

#### Вариант 3 — [[AsyncSequence]] + AsyncStream (очень мощный для событий)

```swift
actor UserEventEmitter {
    private let (stream, continuation) = AsyncStream<UserEvent>.makeStream()
    
    var events: AsyncStream<UserEvent> { stream }
    
    func emit(_ event: UserEvent) {
        continuation.yield(event)
    }
    
    deinit { continuation.finish() }
}

enum UserEvent {
    case loggedIn(User)
    case profileUpdated
    case loggedOut
}

// Использование
let emitter = UserEventEmitter()

Task {
    for await event in emitter.events {
        switch event {
        case .loggedIn(let user):
            print("Пользователь вошёл: \(user.name)")
        case .profileUpdated:
            await reloadProfile()
        case .loggedOut:
            await showLoginScreen()
        }
    }
}
```

**Когда используют AsyncSequence в 2026**:
- сервер-сайд события (SSE, WebSocket)  
- межмодульные события без NotificationCenter  
- TCA / Composable Architecture (эффекты как стримы)

### 4. Визуальная схема Observer в 2026 году

```mermaid
flowchart TD
    UserViewModel[UserViewModel<br>@Observable] -->|изменение name / avatar| SwiftUI[SwiftUI View]
    SwiftUI -->|отрисовка| UserInterface[Пользовательский интерфейс]
    
    APIService[APIService<br>actor] -->|try await fetch| UserViewModel
    
    subgraph "Альтернативы Observer"
        Combine[Combine @Published] -.-> SwiftUI
        AsyncSequence[AsyncSequence / AsyncStream] -.-> ViewModel
        NotificationCenter[NotificationCenter] -.-> LegacyCode[Legacy-код]
    end
```

### 5. Лучшие практики Observer в Swift 2026

- **SwiftUI** → **только** `@Observable` / `@Published`  
- **[[UIKit]]** → `@MainActor` + `@Published` или `AsyncSequence`  
- **TCA / Composable Architecture** → встроенный Observer через reducer и effects  
- **Combine** — используйте только если проект на iOS 13–15 или нужны сложные операторы  
- **NotificationCenter** — почти не используйте в новом коде (только системные события)  
- **AsyncSequence / AsyncStream** — идеально для потоков событий (логи, аналитика, [[WebSocket]])  
- **[[TaskLocal]]** — передача контекста без параметров (request ID, user ID)  
- **Swift 6 strict concurrency** — Observer через `@Observable` / `actor` + `Sendable` = безопасно  
- **Тестирование** — маленькие ViewModel + моки протоколов = быстрые unit-тесты

**Короткий девиз 2026**:
> «Observer в 2026 году — это уже не ручной список подписчиков, а встроенные механизмы языка: `@Observable`, `@Published`, `AsyncSequence`, TCA.  
> Классический GoF Observer почти не используется — его заменили более мощные и безопасные инструменты Swift.»
