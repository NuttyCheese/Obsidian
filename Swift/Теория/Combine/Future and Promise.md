**Future** и **Promise** — это классический паттерн асинхронного программирования, который позволяет работать с операциями, результат которых будет доступен **в будущем** (отложенный результат).

В Swift нативно этот паттерн встроен в **Combine** начиная с iOS 13 / macOS 10.15 под именем `Future`, а с Swift 5.5 (iOS 15 / macOS 12) получил мощную альтернативу в виде **async/await**.

### Сравнение трёх подходов в 2025–2026 годах

| Подход                  | Год появления | Встроен в язык | Сложность кода | Читаемость | Тестируемость | Когда использовать в 2026 |
|-------------------------|---------------|----------------|----------------|------------|---------------|----------------------------|
| **PromiseKit / RxSwift Promise** | 2015–2018     | Нет            | ★★☆☆☆          | ★★★★☆      | ★★★★☆         | legacy-проекты, где уже используется |
| **Combine.Future**      | iOS 13 (2019) | Да             | ★★★☆☆          | ★★★★☆      | ★★★★★         | проекты на Combine + UIKit |
| **async/await + Task**  | Swift 5.5 (2021) | Да          | ★★★★★          | ★★★★★      | ★★★★★         | **новые проекты** и всё, что можно переписать |

### 1. Combine.Future — классика 2019–2023 годов (всё ещё актуально)

```swift
import Combine

enum AuthError: Error {
    case invalidCredentials
    case networkError
}

func login(email: String, password: String) -> Future<User, AuthError> {
    Future { promise in
        // имитация сетевого запроса
        DispatchQueue.global().asyncAfter(deadline: .now() + 1.5) {
            if email == "test@example.com" && password == "123456" {
                promise(.success(User(id: "123", name: "Test User")))
            } else {
                promise(.failure(.invalidCredentials))
            }
        }
    }
}

// Использование в ViewModel
class LoginViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var user: User?
    
    private var cancellables = Set<AnyCancellable>()
    
    func login(email: String, password: String) {
        isLoading = true
        errorMessage = nil
        
        login(email: email, password: password)
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

**Плюсы Combine.Future в 2026**:
- отлично интегрируется в существующий Combine-пайплайн
- поддерживает все операторы: `.map`, `.flatMap`, `.catch`, `.retry`, `.debounce` и т.д.
- легко тестировать с `Just`, `Fail`, `Deferred`

**Минусы**:
- много boilerplate по сравнению с async/await
- нужно вручную управлять `AnyCancellable`

### 2. Современный стандарт 2024–2026: async/await + Task

```swift
@MainActor
class LoginViewModel: ObservableObject {
    
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var user: User?
    
    func login(email: String, password: String) async {
        isLoading = true
        errorMessage = nil
        
        do {
            let user = try await performLogin(email: email, password: password)
            self.user = user
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    private func performLogin(email: String, password: String) async throws -> User {
        try await withCheckedThrowingContinuation { continuation in
            // имитация сетевого запроса
            DispatchQueue.global().asyncAfter(deadline: .now() + 1.5) {
                if email == "test@example.com" && password == "123456" {
                    continuation.resume(returning: User(id: "123", name: "Test User"))
                } else {
                    continuation.resume(throwing: AuthError.invalidCredentials)
                }
            }
        }
    }
}
```

**Вызов из ViewController / SwiftUI**:

```swift
Task {
    await viewModel.login(email: emailField.text ?? "", password: passwordField.text ?? "")
}
```

**Плюсы async/await в 2026**:
- код читается сверху вниз как синхронный
- нет необходимости в `AnyCancellable`
- встроенная обработка ошибок через `try await`
- легко комбинировать с `@MainActor`, `Task.detached`, `TaskGroup`

**Минусы**:
- не так удобно комбинировать несколько асинхронных операций (нужен `async let` или `TaskGroup`)
- сложнее делать сложные трансформации (нет аналогов `combineLatest`, `debounce`)

### Какой подход выбрать в 2026 году

| Ситуация                                      | Рекомендация 2026                                      | Почему |
|-----------------------------------------------|--------------------------------------------------------|--------|
| Новый проект с чистым SwiftUI                 | **async/await + @Observable**                          | минимум boilerplate, нативная поддержка |
| Смешанный UIKit + SwiftUI проект              | **async/await** в сервисах + Combine в ViewModel       | гибкость |
| Проект уже на Combine (много @Published, sink) | Продолжать на Combine + Future                         | миграция дорого стоит |
| Нужно сложное комбинирование потоков          | **Combine** (combineLatest, debounce, flatMap и т.д.)  | мощные операторы |
| Одноразовые операции (логин, загрузка профиля) | **async/await**                                        | проще и чище |
| Тестирование асинхронного кода                | **async/await** + `XCTest` async тесты                 | встроенная поддержка |

### Итог 2026 года

- **Если проект новый или переписываете** → **async/await + Task** — это современный стандарт Apple  
- **Если проект уже на Combine** → продолжайте использовать `Future`, `Publisher`, `AnyCancellable`  
- **Гибрид** (самый частый в 2026): сервисы на async/await, ViewModel на Combine + `@Published`  
- **Future** в Combine всё ещё актуален, но используется реже — в основном для обёртки старых completion-handler API

Удачи с чистым и современным асинхронным кодом в твоём проекте! 🚀