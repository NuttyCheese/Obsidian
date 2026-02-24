**Combine** — это фреймворк реактивного программирования от Apple, представленный в 2019 году вместе с [[iOS]] 13 / macOS 10.15.

Он позволяет работать с **асинхронными потоками данных** (изменения UI, сеть, уведомления, таймеры, события и т.д.) в декларативном стиле, без необходимости вручную управлять коллбэками, делегатами или замыканиями в большинстве случаев.

Combine — это прямой конкурент [[RxSwift]]/RxCocoa, но встроен в систему и глубоко интегрирован с SwiftUI и новыми API Apple.

### Основные концепции Combine (коротко и чётко, 2025–2026)

| Компонент              | Что это простыми словами                                                     | Аналогия из RxSwift       | Самый частый пример в 2026                                                   |
| ---------------------- | ---------------------------------------------------------------------------- | ------------------------- | ---------------------------------------------------------------------------- |
| **[[Publisher]]**      | Источник данных: может испускать значения, ошибки, завершаться               | Observable                | `@Published`, `URLSession.dataTaskPublisher`, `NotificationCenter.publisher` |
| **[[Subscriber]]**     | Тот, кто получает значения и ошибки ([[sink]], assign, receive)              | Observer                  | `.sink`, `.assign(to:)`                                                      |
| **[[Subscription]]**   | Связь между Publisher и Subscriber — управляет жизненным циклом              | Disposable                | `AnyCancellable`                                                             |
| **[[Operator]]**       | Преобразователь потока ([[map]], [[filter]], combineLatest, debounce и т.д.) | Operator                  | `.map`, `.debounce`, `.receive(on:)`                                         |
| **[[AnyCancellable]]** | Токен подписки — отменяет подписку при [[deinit]]                            | DisposeBag (но без мешка) | `.store(in: &cancellables)`                                                  |

### Самый популярный паттерн в [[UIKit]]-приложениях 2026 года ([[MVVM (Model-View-ViewModel) Architecture|MVVM]] + Combine)

```swift
import UIKit
import Combine

// ViewModel
@MainActor
class LoginViewModel: ObservableObject {
    
    @Published var email = ""
    @Published var password = ""
    @Published var isLoginEnabled = false
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    private let authService: AuthService
    
    init(authService: AuthService = .live) {
        self.authService = authService
        
        // Автоматическая валидация формы
        Publishers.CombineLatest($email, $password)
            .map { email, password in
                !email.isEmpty && password.count >= 6 && email.contains("@")
            }
            .assign(to: &$isLoginEnabled)
    }
    
    func login() {
        isLoginEnabled = false  // отключим кнопку на время запроса
        
        authService.login(email: email, password: password)
            .receive(on: DispatchQueue.main)
            .sink { [weak self] completion in
                self?.isLoginEnabled = true
                if case .failure(let error) = completion {
                    self?.errorMessage = error.localizedDescription
                }
            } receiveValue: { [weak self] user in
                self?.errorMessage = nil
                // переход на главный экран
            }
            .store(in: &cancellables)
    }
}

// ViewController
class LoginViewController: UIViewController {
    
    private let viewModel = LoginViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var emailField = UITextField()
    private lazy var passwordField = UITextField()
    private lazy var loginButton = UIButton(type: .system)
    private lazy var errorLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindUI()
    }
    
    private func setupUI() {
        // настройка полей, кнопки, лейбла...
        loginButton.setTitle("Войти", for: .normal)
        loginButton.isEnabled = false
        errorLabel.textColor = .systemRed
    }
    
    private func bindUI() {
        // UITextField → ViewModel
        emailField.textPublisher
            .assign(to: &$viewModel.email)
        
        passwordField.textPublisher
            .assign(to: &$viewModel.password)
        
        // ViewModel → UI
        viewModel.$isLoginEnabled
            .assign(to: \.isEnabled, on: loginButton)
            .store(in: &cancellables)
        
        viewModel.$errorMessage
            .assign(to: \.text, on: errorLabel)
            .store(in: &cancellables)
        
        // Кнопка → login()
        loginButton.tapPublisher
            .sink { [weak self] in
                self?.viewModel.login()
            }
            .store(in: &cancellables)
    }
}
```

### Топ-10 самых полезных операторов Combine в 2026 году (реальная практика)

1. `.map { ... }` — преобразование значения  
2. `.compactMap { $0 }` — map + фильтр nil  
3. `.filter { $0.count > 3 }` — отсечение неподходящих значений  
4. `.debounce(for: .seconds(0.5), scheduler: RunLoop.main)` — задержка перед обработкой (поиск, валидация)  
5. `.receive(on: DispatchQueue.main)` — переключение на главный поток (обязательно для UI)  
6. `.assign(to: &$property)` — прямая привязка к @Published (iOS 14+)  
7. `.sink { ... }` — основной способ подписки  
8. `.combineLatest` — комбинация нескольких потоков (формы: email + пароль)  
9. `.eraseToAnyPublisher()` — скрытие конкретного типа Publisher при возврате из функции  
10. `.catch { Just(fallback) }` — обработка ошибок

### Как начать использовать Combine сегодня (2026)

1. Импортируй `import Combine`
2. Создай ViewModel с `@Published` свойствами
3. Подписывайся через `.sink` и сохраняй в `Set<AnyCancellable>`
4. Для UI-обновлений всегда используй `.receive(on: DispatchQueue.main)`
5. Для сетевых запросов — `URLSession.shared.dataTaskPublisher`
6. Для уведомлений — `NotificationCenter.default.publisher(for:)`
7. Для [[UITextField]] / [[UISwitch]] — используй `.publisher(for: \.text)` / `.publisher(for: \.isOn)`

### Когда в 2026 году лучше НЕ использовать Combine

- Простые сценарии → **async/await + Task** часто проще и чище
- SwiftUI → встроенный `.onChange`, `.task`, `@Observable` покрывают 80% случаев
- Очень большие команды → Zustand / Jotai / Recoil иногда удобнее для кросс-платформы

Но если проект на чистом UIKit или смешанный UIKit + [[SwiftUI]] — **Combine всё ещё отличный выбор** для реактивного MVVM.

**Короткий итог 2026**:
> Combine — мощный фреймворк для реактивного программирования в iOS/macOS.  
> В 2026 году:  
> - основной паттерн — `@Published` + `.sink` / `.assign` + `Set<AnyCancellable>`  
> - идеален для форм, сетевых цепочек, MVVM в UIKit  
> - для SwiftUI часто заменяется `@Observable` + async/await  
> - это **один из самых сильных** инструментов для создания отзывчивого и поддерживаемого кода
