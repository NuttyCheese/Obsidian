## 1. Что такое `EditingChanged`

**`EditingChanged`** — это событие [[UIControl]], которое срабатывает **каждый раз, когда изменяется текст в [[UITextField]] или [[UITextView]]** во время редактирования.

- Используется в **UITextField** и **UITextView** (через UIControl для UITextField)
    
- Отличается от [[EditingDidEnd]], которое срабатывает при завершении редактирования (когда пользователь покидает поле)
    
- Часто применяется для **валидации текста в реальном времени** или обновления интерфейса
    

> Проще говоря: `EditingChanged` = «текст в поле изменился прямо сейчас».

---

## 2. Основные термины

| Термин                        | Описание                                                               |
| ----------------------------- | ---------------------------------------------------------------------- |
| **UIControl.Event**           | Перечисление событий, которые могут происходить с элементами UIControl |
| **EditingChanged**            | Событие изменения текста в текстовом поле                              |
| **Target-Action**             | Механизм связывания события и метода                                   |
| **[[@IBAction]] / addTarget** | Способы обработать событие в коде                                      |
| **UITextField**               | Текстовое поле, которое поддерживает событие EditingChanged            |

---

## 3. Основной синтаксис

### Через @IBAction

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    print("Text changed: \(sender.text ?? "")")
}
```

- В Interface Builder привязывается к событию **Editing Changed**
    

### Через addTarget

```swift
let textField = UITextField()
textField.addTarget(self, action: #selector(textFieldEditingChanged(_:)), for: .editingChanged)

@objc func textFieldEditingChanged(_ sender: UITextField) {
    print("Text changed programmatically: \(sender.text ?? "")")
}
```

- Программный способ отслеживания изменения текста
    

---

## 4. Примеры от простого к сложному

### Пример 1. Логирование текста

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    print("Current text: \(sender.text ?? "")")
}
```

- Выводим текущий текст при каждом изменении
    

---

### Пример 2. Валидация текста

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    if let text = sender.text, text.count > 5 {
        sender.backgroundColor = .green
    } else {
        sender.backgroundColor = .red
    }
}
```

- Меняем цвет фона в зависимости от длины текста
    

---

### Пример 3. Включение кнопки по условию

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    submitButton.isEnabled = !(sender.text?.isEmpty ?? true)
}
@IBOutlet weak var submitButton: UIButton!
```

- Кнопка активна только если поле не пустое
    

---

### Пример 4. Ограничение длины текста

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    if let text = sender.text, text.count > 10 {
        sender.text = String(text.prefix(10))
    }
}
```

- Ограничиваем количество символов в реальном времени
    

---

### Пример 5. Форматирование текста

```swift
@IBAction func textFieldEditingChanged(_ sender: UITextField) {
    if let text = sender.text {
        sender.text = text.uppercased() // Преобразуем текст в верхний регистр сразу при вводе
    }
}
```

- Можно применять любое форматирование текста на лету
    

---

## 5. Особенности EditingChanged

1. Срабатывает **каждый раз при изменении текста**, пока пользователь редактирует
    
2. Отличается от **EditingDidEnd** (срабатывает при завершении редактирования)
    
3. Используется для:
    
    - Валидации ввода
        
    - Форматирования текста
        
    - Управления состоянием кнопок и других UI элементов
        
4. Поддерживается как **@IBAction**, так и **addTarget**
    

---

## 6. Итог

- **EditingChanged** = событие изменения текста в UITextField
    
- Позволяет:
    
    - Следить за текстом в реальном времени
        
    - Делать валидацию и форматирование на лету
        
    - Управлять UI элементами в зависимости от введённого текста
        
- Поддерживается **Interface Builder** и **код**
    

---
