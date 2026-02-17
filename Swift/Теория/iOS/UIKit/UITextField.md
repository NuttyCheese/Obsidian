**`UITextField`** — это **однострочное текстовое поле**, которое позволяет пользователю:

- Вводить текст
    
- Редактировать текст
    
- Получать уведомления о начале и окончании редактирования
    

Особенности:

- Используется для **логина, пароля, поиска, ввода чисел**
    
- Может быть настроено:
    
    - Плейсхолдер (`placeholder`)
        
    - Клавиатура (`keyboardType`)
        
    - Текст, цвет текста, стиль шрифта
        
- Поддерживает **делегаты ([[UITextFieldDelegate]])** для обработки событий
    

> Проще говоря: `UITextField` = «поле ввода текста пользователем».

---

## 2. Основные термины

| Термин                  | Описание                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **text**                | Текущий текст в поле                                                                 |
| **placeholder**         | Текст-подсказка до ввода                                                             |
| **keyboardType**        | Тип клавиатуры (`.default`, `.numberPad`, `.emailAddress` и др.)                     |
| **isSecureTextEntry**   | Если true, текст скрыт (например, для пароля)                                        |
| **delegate**            | Делегат, который реагирует на события текстового поля                                |
| **UITextFieldDelegate** | Протокол для обработки событий начала/окончания редактирования, нажатий Return и др. |
| **addTarget**           | Можно привязать событие [[EditingChanged]] или [[EditingDidEnd]]                     |

---

## 3. Основной синтаксис

### Создание UITextField программно

```swift
let textField = UITextField(frame: CGRect(x: 50, y: 200, width: 300, height: 40))
textField.placeholder = "Введите имя"
textField.borderStyle = .roundedRect
textField.textColor = .black
textField.font = UIFont.systemFont(ofSize: 16)
textField.keyboardType = .default
textField.addTarget(self, action: #selector(textFieldChanged(_:)), for: .editingChanged)
view.addSubview(textField)

@objc func textFieldChanged(_ sender: UITextField) {
    print("Текст изменился: \(sender.text ?? "")")
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейшее текстовое поле

```swift
let textField = UITextField()
textField.placeholder = "Введите текст"
view.addSubview(textField)
```

- Просто создаём поле ввода
    

---

### Пример 2. Обработка события изменения текста

```swift
textField.addTarget(self, action: #selector(textChanged(_:)), for: .editingChanged)

@objc func textChanged(_ sender: UITextField) {
    print("Текст сейчас: \(sender.text ?? "")")
}
```

- Реагируем на каждое изменение текста
    

---

### Пример 3. Делегат UITextFieldDelegate

```swift
class ViewController: UIViewController, UITextFieldDelegate {
    override func viewDidLoad() {
        super.viewDidLoad()
        let textField = UITextField(frame: CGRect(x: 50, y: 200, width: 300, height: 40))
        textField.delegate = self
        view.addSubview(textField)
    }

    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder() // скрываем клавиатуру
        return true
    }
}
```

- Используем делегат для обработки нажатия Return
    

---

### Пример 4. Парольное поле

```swift
let passwordField = UITextField(frame: CGRect(x: 50, y: 250, width: 300, height: 40))
passwordField.isSecureTextEntry = true
passwordField.placeholder = "Введите пароль"
passwordField.borderStyle = .roundedRect
view.addSubview(passwordField)
```

- Текст скрыт точками для безопасности
    

---

### Пример 5. Поле с клавиатурой для чисел и проверкой

```swift
let numberField = UITextField(frame: CGRect(x: 50, y: 300, width: 300, height: 40))
numberField.keyboardType = .numberPad
numberField.placeholder = "Введите число"
numberField.borderStyle = .roundedRect
numberField.addTarget(self, action: #selector(numberChanged(_:)), for: .editingChanged)
view.addSubview(numberField)

@objc func numberChanged(_ sender: UITextField) {
    if let text = sender.text, let number = Int(text) {
        print("Введено число: \(number)")
    } else {
        print("Введите корректное число")
    }
}
```

- Поле принимает только числа и проверяет ввод
    

---

## 5. Особенности UITextField

1. Поддерживает **однострочный ввод**
    
2. Можно использовать **плейсхолдер** и **стиль рамки**
    
3. Делегат позволяет:
    
    - Управлять началом и окончанием редактирования
        
    - Реагировать на Return
        
    - Ограничивать ввод символов
        
4. `addTarget` позволяет реагировать на изменения текста
    
5. Используется для: логина, пароля, поиска, ввода чисел
    

---

## 6. Итог

- **UITextField** = однострочное текстовое поле для ввода пользователем
    
- Позволяет:
    
    - Отслеживать изменения текста
        
    - Настраивать вид и клавиатуру
        
    - Реагировать на события через делегат или target-action
        
- Отличие от [[UITextView]]: **однострочное поле, не многострочный текст**
    

---
