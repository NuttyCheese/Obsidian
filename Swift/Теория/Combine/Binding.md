**Binding в Combine** — это не отдельный тип (как в [[SwiftUI]]), а **механизм двусторонней синхронизации** между источником данных и UI-элементом, который достигается с помощью **Publisher** + **Subscriber** + **.assign(to:)** или **.sink**.

В UIKit Combine не даёт готового типа `Binding`, как в SwiftUI, поэтому мы строим его вручную.  
Вот **самые актуальные и рекомендуемые** способы в 2025–2026 годах.

### 1. Классический двусторонний Binding ([[UIKit]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

// ViewModel
class FormViewModel: ObservableObject {
    @Published var username: String = ""
    @Published var isValid: Bool = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // Валидация в реальном времени
        $username
            .map { $0.count >= 3 }
            .assign(to: &$isValid)
    }
}

// ViewController
class FormViewController: UIViewController {
    
    private let viewModel = FormViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var textField: UITextField = {
        let tf = UITextField()
        tf.borderStyle = .roundedRect
        tf.placeholder = "Имя пользователя"
        tf.translatesAutoresizingMaskIntoConstraints = false
        return tf
    }()
    
    private lazy var validationLabel: UILabel = {
        let lbl = UILabel()
        lbl.textColor = .systemRed
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindUI()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        view.addSubview(textField)
        view.addSubview(validationLabel)
        
        NSLayoutConstraint.activate([
            textField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 100),
            textField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 40),
            textField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -40),
            textField.heightAnchor.constraint(equalToConstant: 44),
            
            validationLabel.topAnchor.constraint(equalTo: textField.bottomAnchor, constant: 16),
            validationLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    private func bindUI() {
        // 1. Из UI → ViewModel (изменения текста → @Published)
        textField.publisher(for: \.text)
            .compactMap { $0 }
            .assign(to: &$viewModel.username)
        
        // 2. Из ViewModel → UI (изменения @Published → обновление label)
        viewModel.$isValid
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isValid in
                self?.validationLabel.text = isValid ? "Имя валидно ✓" : "Минимум 3 символа"
                self?.validationLabel.textColor = isValid ? .systemGreen : .systemRed
            }
            .store(in: &cancellables)
        
        // 3. (Опционально) отложенная валидация с debounce
        viewModel.$username
            .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
            .sink { [weak self] _ in
                print("Пользователь закончил ввод:", self?.viewModel.username ?? "")
            }
            .store(in: &cancellables)
    }
}
```

### 2. Самый современный и чистый способ ([[iOS]] 14+): assign(to:) + $property

```swift
// В ViewModel
@Published var email = ""
@Published var password = ""
@Published var isLoginEnabled = false

init() {
    Publishers.CombineLatest($email, $password)
        .map { email, password in
            !email.isEmpty && password.count >= 6
        }
        .assign(to: &$isLoginEnabled)
}
```

В контроллере:

```swift
viewModel.$isLoginEnabled
    .receive(on: DispatchQueue.main)
    .assign(to: \.isEnabled, on: loginButton)
    .store(in: &cancellables)
```

### 3. Расширения для удобства (очень популярны в 2026)

```swift
extension UITextField {
    var textPublisher: AnyPublisher<String, Never> {
        NotificationCenter.default.publisher(for: UITextField.textDidChangeNotification, object: self)
            .compactMap { ($0.object as? UITextField)?.text }
            .eraseToAnyPublisher()
    }
}

extension UISwitch {
    var isOnPublisher: AnyPublisher<Bool, Never> {
        publisher(for: \.isOn)
            .eraseToAnyPublisher()
    }
}

extension UIButton {
    var tapPublisher: AnyPublisher<Void, Never> {
        publisher(for: .touchUpInside)
            .map { _ in () }
            .eraseToAnyPublisher()
    }
}
```

Использование:

```swift
textField.textPublisher
    .assign(to: &$viewModel.username)

switchControl.isOnPublisher
    .assign(to: &$viewModel.isDarkMode)

button.tapPublisher
    .sink { [weak self] in
        self?.viewModel.login()
    }
    .store(in: &cancellables)
```

### 4. Полный пример с валидацией формы и кнопкой входа

```swift
class LoginViewModel: ObservableObject {
    @Published var email = ""
    @Published var password = ""
    @Published var isLoginEnabled = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        Publishers.CombineLatest($email, $password)
            .map { email, password in
                email.contains("@") && password.count >= 6
            }
            .assign(to: &$isLoginEnabled)
    }
    
    func login() {
        print("Попытка входа: \(email), пароль: \(password)")
        // здесь вызов API
    }
}

class LoginViewController: UIViewController {
    
    private let viewModel = LoginViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var emailField = UITextField()
    private lazy var passwordField = UITextField()
    private lazy var loginButton = UIButton(type: .system)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bind()
    }
    
    private func setupUI() {
        // ... настройка полей и кнопки
        loginButton.setTitle("Войти", for: .normal)
        loginButton.isEnabled = false
    }
    
    private func bind() {
        emailField.textPublisher
            .assign(to: &$viewModel.email)
        
        passwordField.textPublisher
            .assign(to: &$viewModel.password)
        
        viewModel.$isLoginEnabled
            .assign(to: \.isEnabled, on: loginButton)
            .store(in: &cancellables)
        
        loginButton.tapPublisher
            .sink { [weak self] in
                self?.viewModel.login()
            }
            .store(in: &cancellables)
    }
}
```

### Лучшие практики Combine Binding в UIKit 2026

- **Всегда** храните подписки в `Set<AnyCancellable>` или `[AnyCancellable]`  
- **Используйте** `assign(to:)` для прямой привязки `@Published` → UI-свойство  
- **Для [[UITextField]]** — добавьте расширение `textPublisher`  
- **Для debounce / throttle** — используйте перед `.assign` или `.sink`  
- **Для сложных форм** — `CombineLatest` или `Publishers.Zip`  
- **Для SwiftUI** — используйте `@Binding` и `.onChange` — Combine не обязателен  
- **Документируйте** — пишите комментарий «Двусторонний Binding: UITextField <-> @Published username через Combine»

**Короткий итог 2026**:
> Binding в UIKit + Combine — это **двусторонняя синхронизация** UI ↔ ViewModel через `.publisher(for:)` + `.assign(to:)` + `.sink`.  
> В 2026 году:  
> - храните подписки в `Set<AnyCancellable>`  
> - используйте расширения для `textPublisher`, `isOnPublisher` и т.д.  
> - для форм — `CombineLatest` + `debounce`  
> - это **самый чистый** и **самый надёжный** способ MVVM в UIKit с Combine  
