**UITextFieldDelegate** — это протокол в [[UIKit]], который позволяет полностью контролировать поведение **[[UITextField]]** (однострочное текстовое поле ввода).

Он отвечает за:
- разрешение / запрет начала и окончания редактирования
- фильтрацию и ограничение вводимых символов
- обработку нажатия клавиши [[Return]]
- реакцию на изменение текста и выделения
- управление фокусом и клавиатурой

Это **один из самых часто используемых** делегатов в UIKit-приложениях 2026 года, особенно в формах, логине, поиске, вводе номеров, email, паролей и любых местах, где нужен точный контроль над однострочным текстом.

### Основные методы UITextFieldDelegate (самые актуальные в 2026)

| Метод делегата                                      | Когда вызывается                                                                 | Что обычно делают внутри метода                                      | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------|
| `textFieldShouldBeginEditing(_:)`                   | Перед тем, как поле станет первым респондером (появится клавиатура)            | Разрешить/запретить фокус, показать/скрыть клавиатуру                 | Запрет редактирования в режиме readonly |
| `textFieldDidBeginEditing(_:)`                      | После того, как поле стало первым респондером                                   | Анимация, показ placeholder → скрыть, установка курсора               | Скрытие placeholder, выделение текста |
| `textFieldShouldEndEditing(_:)`                     | Перед тем, как поле потеряет фокус                                              | Валидация, сохранение черновика, запрет потери фокуса                 | Валидация перед уходом с поля |
| `textFieldDidEndEditing(_:)`                        | После потери фокуса                                                             | Сохранение значения, валидация, скрытие клавиатуры                    | Сохранение введённого email/пароля |
| `textField(_:shouldChangeCharactersIn:replacementString:)` | Перед каждым изменением текста (вставка, удаление, paste)                  | Ограничение длины, фильтрация символов, форматирование в реальном времени | Только цифры, маска телефона, лимит 30 символов |
| `textFieldShouldReturn(_:)`                         | При нажатии клавиши Return                                                      | Скрыть клавиатуру, перейти к следующему полю, отправить форму         | Переход к следующему полю или отправка |
| `textFieldDidChangeSelection(_:)`                   | При изменении позиции курсора или выделения текста                              | Реакция на выделение (форматирование, контекстное меню)               | Кастомный toolbar при выделении |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UITextField]] + UITextFieldDelegate + [[Combine]] + маска + валидация)

```swift
import UIKit
import Combine

class LoginViewController: UIViewController, UITextFieldDelegate {
    
    private let viewModel = LoginViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var emailTextField: UITextField = {
        let tf = UITextField()
        tf.placeholder = "Email"
        tf.keyboardType = .emailAddress
        tf.autocapitalizationType = .none
        tf.autocorrectionType = .no
        tf.borderStyle = .roundedRect
        tf.clearButtonMode = .whileEditing
        tf.delegate = self
        tf.translatesAutoresizingMaskIntoConstraints = false
        return tf
    }()
    
    private lazy var passwordTextField: UITextField = {
        let tf = UITextField()
        tf.placeholder = "Пароль"
        tf.isSecureTextEntry = true
        tf.borderStyle = .roundedRect
        tf.delegate = self
        tf.translatesAutoresizingMaskIntoConstraints = false
        return tf
    }()
    
    private lazy var loginButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Войти", for: .normal)
        btn.isEnabled = false
        btn.translatesAutoresizingMaskIntoConstraints = false
        return btn
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        let stack = UIStackView(arrangedSubviews: [emailTextField, passwordTextField, loginButton])
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 32),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -32)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Текст → ViewModel
        emailTextField.publisher(for: \.text)
            .removeDuplicates()
            .assign(to: \.email, on: viewModel)
            .store(in: &cancellables)
        
        passwordTextField.publisher(for: \.text)
            .removeDuplicates()
            .assign(to: \.password, on: viewModel)
            .store(in: &cancellables)
        
        // Валидация → кнопка
        viewModel.$isLoginEnabled
            .receive(on: DispatchQueue.main)
            .assign(to: \.isEnabled, on: loginButton)
            .store(in: &cancellables)
    }
    
    // MARK: - UITextFieldDelegate
    
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        if textField == emailTextField {
            passwordTextField.becomeFirstResponder()
        } else if textField == passwordTextField {
            textField.resignFirstResponder()
            // Попытка логина
            viewModel.login()
        }
        return true
    }
    
    func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        if textField == emailTextField {
            // Ограничение длины email (пример)
            let maxLength = 64
            let currentText = (textField.text ?? "") as NSString
            let newText = currentText.replacingCharacters(in: range, with: string)
            return newText.count <= maxLength
        }
        
        if textField == passwordTextField {
            // Запрещаем пробелы в пароле
            return !string.contains(" ")
        }
        
        return true
    }
    
    func textFieldDidBeginEditing(_ textField: UITextField) {
        // Можно подсветить поле
        textField.layer.borderColor = UIColor.systemBlue.cgColor
    }
    
    func textFieldDidEndEditing(_ textField: UITextField) {
        textField.layer.borderColor = UIColor.systemGray4.cgColor
        // Валидация при потере фокуса
    }
}

// ViewModel
@MainActor
class LoginViewModel: ObservableObject {
    @Published var email: String = ""
    @Published var password: String = ""
    @Published var isLoginEnabled = false
    
    init() {
        $email.combineLatest($password)
            .map { email, password in
                !email.isEmpty &&
                email.contains("@") &&
                password.count >= 6
            }
            .assign(to: &$isLoginEnabled)
    }
    
    func login() {
        // Здесь логика авторизации
    }
}
```

### Лучшие практики UITextFieldDelegate в 2026 году

- **Слабая ссылка** — делайте `weak var delegate: UITextFieldDelegate?` в кастомных UITextField  
- **Ограничение ввода** — реализуйте в `shouldChangeCharactersIn` — это самый надёжный способ  
- **Переход по Return** — используйте `textFieldShouldReturn` для фокуса на следующем поле или отправки формы  
- **Для масок** — форматируйте текст в `shouldChangeCharactersIn` (телефон, карта, код)  
- **Для Combine** — используйте `publisher(for: \.text)` — это самый удобный способ двусторонней привязки  
- **Для [[SwiftUI]]** — используйте `TextField` — UITextFieldDelegate нужен только в UIKit  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint`  
- **Документируйте** — пишите комментарий:

```swift
extension LoginViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        // Переход к следующему полю или отправка
    }
}
```

**Короткий итог 2026**:
> `UITextFieldDelegate` — протокол для **полного контроля** над однострочным текстовым полем UITextField.  
> В 2026 году:  
> - ключевые методы — `shouldChangeCharactersIn`, `textFieldShouldReturn`, `textFieldDidBegin/EndEditing`  
> - самый популярный паттерн — маска, ограничение длины, переход по Return, валидация + Combine  
> - идеален для email, пароля, телефона, поиска, логина  
> - в SwiftUI — эквивалент `TextField` с `.onChange` и `.focused`  
> - это **фундаментальный** делегат для любой формы ввода в UIKit  
