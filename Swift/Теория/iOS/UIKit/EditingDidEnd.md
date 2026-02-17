**`EditingDidEnd`** — это событие типа `UIControl.Event.editingDidEnd`, которое срабатывает **один раз**, когда пользователь **завершает редактирование** текста в [[UITextField]].

### Когда именно происходит EditingDidEnd (2026 актуально)

| Действие пользователя                          | Срабатывает EditingDidEnd? | Примечание |
|------------------------------------------------|-----------------------------|------------|
| Нажал **Return** на клавиатуре                 | **Да**                      | Самый частый триггер |
| Тапнул за пределы поля (на другой элемент)     | **Да**                      | Стандартное поведение |
| Переключился на другое текстовое поле          | **Да**                      | При переходе фокуса |
| Программно вызвал `resignFirstResponder()`     | **Да**                      | То же самое |
| Закрыл клавиатуру (системная кнопка «Готово»)  | **Да**                      | — |
| Поле потеряло фокус из-за системного события (звонок, уведомление) | **Да** (если редактирование было активно) | — |
| Пользователь просто смотрит, но не редактирует | **Нет**                     | — |

### Чем отличается от [[EditingChanged]]

| Событие          | Когда срабатывает                      | Кол-во вызовов при вводе «Hello» | Самый частый сценарий 2026                                       |
| ---------------- | -------------------------------------- | -------------------------------- | ---------------------------------------------------------------- |
| `editingChanged` | Каждый символ, вставка, удаление       | 5 раз                            | Live-валидация, поиск в реальном времени, форматирование на лету |
| `editingDidEnd`  | Только после завершения редактирования | 1 раз                            | Финальная валидация, сохранение, переход к следующему полю       |

### Когда использовать именно `EditingDidEnd` в 2026 году

| Сценарий                                               | Почему именно `EditingDidEnd`                                | Пример кода (коротко)                                 |
| ------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| Финальная проверка email / пароля                      | Проверяем только после того, как пользователь закончил ввод  | `if !isValidEmail(text) { showError() }`              |
| Сохранение значения в [[Swift/Теория/Хранение данных/UserDefaults]] / [[Core Data]] | Не нужно сохранять каждый символ — только финальное значение | `UserDefaults.standard.set(text, forKey: "username")` |
| Переход к следующему полю ввода                        | Автоматический focus на следующее поле после Return          | `nextTextField.becomeFirstResponder()`                |
| Отправка формы по Enter                                | Запуск логина/регистрации/поиска после нажатия [[return]]    | `login()`                                             |
| Обновление UI после ввода                              | Показать/скрыть кнопку «Далее» только после завершения       | `nextButton.isEnabled = !text.isEmpty`                |
| Логирование / аналитика ввода                          | Отправить событие только один раз, а не на каждый символ     | `Analytics.logEvent("username_entered")`              |

### Самый современный паттерн 2026 года (с [[@IBAction]] + addTarget)

```swift
final class LoginViewController: UIViewController {
    
    @IBOutlet weak var emailTextField: UITextField!
    @IBOutlet weak var passwordTextField: UITextField!
    @IBOutlet weak var loginButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Вариант 1: через IBAction (подключено в Storyboard/XIB)
        // emailTextField.addTarget(self, action: #selector(editingDidEnd), for: .editingDidEnd)
        
        // Вариант 2: программно (если нет IB)
        [emailTextField, passwordTextField].forEach {
            $0.addTarget(self, action: #selector(editingDidEnd), for: .editingDidEnd)
        }
        
        updateLoginButtonState()
    }
    
    @objc private func editingDidEnd(_ textField: UITextField) {
        guard let text = textField.text else { return }
        
        switch textField {
        case emailTextField:
            validateEmail(text)
        case passwordTextField:
            validatePassword(text)
        default:
            break
        }
        
        updateLoginButtonState()
    }
    
    private func validateEmail(_ text: String) {
        let isValid = text.contains("@") && text.contains(".")
        emailTextField.textColor = isValid ? .label : .systemRed
        
        if !isValid {
            showError("Некорректный email")
        }
    }
    
    private func validatePassword(_ text: String) {
        let isValid = text.count >= 6
        passwordTextField.textColor = isValid ? .label : .systemRed
    }
    
    private func updateLoginButtonState() {
        let email = emailTextField.text ?? ""
        let password = passwordTextField.text ?? ""
        
        loginButton.isEnabled = !email.isEmpty && password.count >= 6
        loginButton.backgroundColor = loginButton.isEnabled ? .systemBlue : .systemGray
    }
    
    private func showError(_ message: String) {
        let alert = UIAlertController(title: "Ошибка", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

### Лучшие практики EditingDidEnd в Swift 2026

- **Не делай тяжёлую работу** — событие вызывается один раз, но если пользователь быстро переключается между полями — может быть много вызовов  
- **resignFirstResponder()** — можно вызывать здесь для автоматического скрытия клавиатуры  
- **nextField.becomeFirstResponder()** — стандартный паттерн для перехода по Return  
- **debounce** — если нужна тяжёлая валидация (запрос на сервер), добавляй задержку 0.3–0.5 сек  
- **[[@MainActor]]** — все обработчики UI-событий — на главном акторе  
- **[[Swift]] 6 strict concurrency** — событие вызывается на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «@objc func — финальная валидация и сохранение после завершения редактирования»

**Короткий девиз 2026**:
> `EditingDidEnd` — это когда тебе нужно **обработать финальный текст** после того, как пользователь закончил ввод: проверить, сохранить, перейти дальше, отправить форму.  
> В 2026 году это **основное** событие для финальной валидации и действий после редактирования.  
> Для live-изменений — используй `editingChanged`, для финала — `editingDidEnd`.
