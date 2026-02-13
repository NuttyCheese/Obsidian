**Swift Concurrency** — это современная модель конкурентного программирования, представленная Apple в 2021 году ([[Swift]] 5.5) и значительно доработанная к 2026 году (Swift 6+).

Она радикально упростила написание многопоточного кода, сделав его **безопасным**, **читаемым** и **производительным** по сравнению с GCD и OperationQueue.

### Основные строительные блоки Swift Concurrency (2026)

| Компонент                         | Что это простыми словами                               | Главное преимущество в 2026 году                          | Когда использовать (рекомендация)    |
| --------------------------------- | ------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------ |
| [[async]] / [[await]]             | «Подожди результат, не блокируя поток»                 | Самый читаемый и естественный синтаксис                   | 95% асинхронного кода                |
| [[Task]]                          | «Запусти асинхронную работу»                           | Основная единица асинхронной работы                       | Запуск любых асинхронных операций    |
| [[actor]]                         | «Класс с автоматической защитой от гонок данных»       | Самый безопасный способ хранения изменяемого состояния    | Почти всё изменяемое состояние       |
| [[@MainActor]]                    | «Этот код должен выполняться только на главном потоке» | Заменил [[DispatchQueue]].[[main]] в 90% случаев          | ViewModel, UI-обновления             |
| [[TaskGroup]] / `withTaskGroup`   | «Запусти много задач параллельно и дождись всех»       | Структурированная конкурентность                          | Параллельные загрузки, map-reduce    |
| [[AsyncSequence]] / [[for-await]] | «Асинхронный аналог Sequence»                          | Потоки событий, стримы, WebSocket, [[NotificationCenter]] | SSE, [[WebSocket]], таймеры, события |
| [[TaskLocal]]                     | «Локальные переменные для каждой Task»                 | Передача контекста (request ID, user ID) без параметров   | Трассировка, логирование, auth       |
| [[Sendable]]                      | «Этот тип безопасно передавать между задачами»         | Защита от [[data race]] на уровне компилятора             | Всё, что передаётся между задачами   |

### Самые популярные идиоматичные паттерны 2026 года

#### 1. Загрузка данных + обновление UI (самый частый сценарий)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
    
    private let repository: any UserRepository
    
    init(repository: any UserRepository) {
        self.repository = repository
    }
    
    func load() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            user = try await repository.fetchCurrentUser()
        } catch {
            // обработка ошибки
        }
    }
}

// В SwiftUI
struct ProfileView: View {
    @State private var vm = ProfileViewModel(repository: APIRepository())
    
    var body: some View {
        Group {
            if let user = vm.user {
                Text(user.name)
            } else if vm.isLoading {
                ProgressView()
            } else {
                Text("Ошибка загрузки")
            }
        }
        .task { await vm.load() }
    }
}
```

#### 2. Параллельная загрузка нескольких ресурсов

```swift
func loadProfileData() async throws -> (User, [Post], UIImage?) {
    async let user = repository.fetchCurrentUser()
    async let posts = repository.fetchPosts()
    async let avatar = imageLoader.load(from: user.avatarURL)
    
    return try await (user, posts, avatar)
}
```

#### 3. Actor — современная замена [[NSLock|lock]] / dispatch_once / [[singleton]]

```swift
actor Cache {
    private var storage: [URL: Data] = [:]
    
    func save(_ data: Data, for url: URL) {
        storage[url] = data
    }
    
    func get(for url: URL) -> Data? {
        storage[url]
    }
}

let cache = Cache()

Task {
    let data = try await downloadImage(from: url)
    await cache.save(data, for: url)
}
```

#### 4. TaskLocal — передача контекста без параметров

```swift
@TaskLocal static var requestID: String?

func handleRequest() async {
    let id = UUID().uuidString
    
    await TaskLocal.requestID.withValue(id) {
        await processRequest()     // requestID доступен во всех вложенных вызовах
        await logMetrics()         // тоже видит requestID
    }
}
```

### 5. Сравнение Swift Concurrency vs [[GCD]] (2026)

| Критерий                             | Swift Concurrency (Task / actor / async) | GCD (DispatchQueue.[[global]]) | Победитель 2026 |
| ------------------------------------ | ---------------------------------------- | ------------------------------ | --------------- |
| Читаемость                           | ★★★★★                                    | ★★★☆☆                          | Concurrency     |
| Безопасность от data race            | ★★★★★ (компилятор + actor)               | ★★☆☆☆ (ручная)                 | Concurrency     |
| Отмена задач                         | ★★★★★ (`task.cancel()`)                  | ★☆☆☆☆                          | Concurrency     |
| Асинхронность без callback           | ★★★★★ (await)                            | ★★☆☆☆                          | Concurrency     |
| Производительность в горячих петлях  | ★★★★☆                                    | ★★★★★                          | GCD (редко)     |
| Поддержка Swift 6 strict concurrency | Полная                                   | Частичная                      | Concurrency     |
| Общий рейтинг в новых проектах       | 95%+                                     | 5%                             | Concurrency     |

### 6. Лучшие практики Swift Concurrency 2026

- **actor** — для **всего изменяемого состояния** (заменил 90% lock / dispatch_once / singleton)  
- **@MainActor** — для **всего UI и ViewModel**  
- **Task.detached** — только для тяжёлых вычислений без наследования контекста  
- **TaskGroup / withTaskGroup** — для динамического параллелизма  
- **AsyncSequence / for await** — для потоков событий (WebSocket, SSE, NotificationCenter)  
- **TaskLocal** — для передачи контекста (request ID, trace ID, user ID) без параметров  
- **Sendable** — всё, что передаётся между задачами, должно быть Sendable  
- **Не используй GCD в новом коде**, кроме очень специфичных случаев (barrier в concurrent queue, тонкая настройка QoS)

**Короткий девиз 2026**:
> «Swift Concurrency в 2026 году — это когда ты пишешь многопоточный код так же просто, как однопоточный.  
> actor, async/await, Task — это новый стандарт.  
> GCD — legacy-инструмент для поддержки старого кода.»
