**`EditingChanged`** — это событие типа `UIControl.Event`, которое срабатывает **каждый раз, когда текст в `UITextField` изменяется** во время активного редактирования (пользователь печатает, стирает, вставляет, автокоррект меняет символ и т.д.).

### Ключевые отличия от похожих событий (2026 актуально)

| Событие                  | Когда срабатывает                                      | Самый частый сценарий использования 2026 |
|--------------------------|--------------------------------------------------------|------------------------------------------|
| `editingChanged`         | Каждый символ при вводе/удалении/вставке               | Live-валидация, форматирование, поиск в реальном времени |
| `editingDidBegin`        | Пользователь начал редактировать (тапнул в поле)       | Показать клавиатуру, подсветить поле    |
| `editingDidEnd`          | Пользователь закончил редактирование (тап за пределы)  | Сохранить значение, скрыть клавиатуру    |
| `editingDidEndOnExit`    | Завершение редактирования по нажатию Return            | Отправка формы по Enter                  |
| `valueChanged`           | Для UISlider, UISwitch, UISegmentedControl и т.д.      | Не для текста                            |

### Когда использовать `editingChanged` в 2026 году

| Сценарий                                      | Почему именно `editingChanged`                         | Пример кода (коротко) |
|-----------------------------------------------|--------------------------------------------------------|-----------------------|
| Live-валидация пароля / email                 | Показать зелёный/красный фон или иконку сразу          | `if text.count >= 8 { green } else { red }` |
| Форматирование номера телефона / карты        | Автоматически добавлять пробелы/дефисы при вводе       | `text = formatPhone(text)` |
| Поиск в реальном времени                      | Фильтровать список по мере ввода текста                 | `tableView.reloadData()` на каждый символ |
| Подсчёт символов / ограничение длины          | Показывать «Осталось 120/150» или обрезать текст       | `label.text = "\(text.count)/150"` |
| Активация кнопки «Далее» / «Отправить»        | Кнопка enabled только если поле заполнено корректно    | `button.isEnabled = !text.isEmpty` |
| Маски / автодополнение                        | Показывать подсказки или форматировать на лету         | `text = maskCreditCard(text)` |

### Самый современный паттерн 2026 года

```swift
final class LoginViewController: UIViewController {
    
    @IBOutlet weak var emailTextField: UITextField!
    @IBOutlet weak var passwordTextField: UITextField!
    @IBOutlet weak var loginButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Вариант 1: через IBAction в Interface Builder
        // (подключаем событие Editing Changed в Storyboard/XIB)
        
        // Вариант 2: программно (если нет IB)
        emailTextField.addTarget(self, action: #selector(textFieldEditingChanged), for: .editingChanged)
        passwordTextField.addTarget(self, action: #selector(textFieldEditingChanged), for: .editingChanged)
        
        updateLoginButtonState()
    }
    
    @objc private func textFieldEditingChanged(_ textField: UITextField) {
        updateLoginButtonState()
        
        // Дополнительная логика в зависимости от поля
        switch textField {
        case emailTextField:
            validateEmail(textField.text ?? "")
        case passwordTextField:
            validatePassword(textField.text ?? "")
        default:
            break
        }
    }
    
    private func updateLoginButtonState() {
        let emailValid = !(emailTextField.text?.isEmpty ?? true)
        let passwordValid = (passwordTextField.text?.count ?? 0) >= 6
        
        loginButton.isEnabled = emailValid && passwordValid
        loginButton.backgroundColor = loginButton.isEnabled ? .systemBlue : .systemGray
    }
    
    private func validateEmail(_ text: String) {
        // Простая валидация в реальном времени
        let isValid = text.contains("@") && text.contains(".")
        emailTextField.textColor = isValid ? .label : .systemRed
    }
    
    private func validatePassword(_ text: String) {
        passwordTextField.textColor = text.count >= 6 ? .label : .systemRed
    }
}
```

### Лучшие практики `EditingChanged` в Swift 2026

- **debounce** — если логика тяжёлая (поиск по сети, сложная валидация) — добавляй задержку 0.3–0.5 сек

```swift
private var searchWorkItem: DispatchWorkItem?
@objc func textFieldEditingChanged(_ textField: UITextField) {
    searchWorkItem?.cancel()
    let workItem = DispatchWorkItem { [weak self] in
        self?.performSearch(textField.text ?? "")
    }
    searchWorkItem = workItem
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.4, execute: workItem)
}
```

- **Не делай тяжёлые операции** в обработчике — это вызывается на каждый символ  
- **Используй `textField(_:shouldChangeCharactersIn:replacementString:)`** — если нужно ограничить ввод (только цифры, максимум символов)  
- **@MainActor** — все обработчики событий UI — на главном акторе  
- **Swift 6 strict concurrency** — `editingChanged` вызывается на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «@objc func — live-валидация и активация кнопки при изменении текста»

**Короткий девиз 2026**:
> `EditingChanged` — это когда тебе нужно **реагировать на каждый введённый символ** в текстовом поле: валидация в реальном времени, форматирование, активация кнопки, поиск по мере ввода.  
> В 2026 году это **основной** способ сделать текстовые поля «живыми» и отзывчивыми.  
> Для тяжёлой логики — добавляй debounce.

Удачи с мгновенной и удобной обработкой текста в Swift! ⌨️