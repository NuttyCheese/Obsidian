## 1. Что такое `EditingDidEnd`

**`EditingDidEnd`** — это событие [[UIControl]], которое срабатывает **когда пользователь завершает редактирование текста в [[UITextField]]**.

- Отличается от **EditingChanged**, которое срабатывает при каждом изменении текста
    
- Срабатывает, когда пользователь:
    
    - Нажимает на кнопку «Return» на клавиатуре
        
    - Переходит в другое текстовое поле
        
    - Закрывает клавиатуру
        
- Часто используется для **валидации текста или запуска действий после ввода**
    

> Проще говоря: `EditingDidEnd` = «пользователь закончил ввод текста».

---

## 2. Основные термины

| Термин                        | Описание                                                               |
| ----------------------------- | ---------------------------------------------------------------------- |
| **UIControl.Event**           | Перечисление событий, которые могут происходить с элементами UIControl |
| **EditingDidEnd**             | Событие завершения редактирования текста в UITextField                 |
| **Target-Action**             | Механизм связывания события с методом                                  |
| **[[@IBAction]] / addTarget** | Способы обработать событие в коде                                      |
| **UITextField**               | Текстовое поле, которое поддерживает событие EditingDidEnd             |

---

## 3. Основной синтаксис

### Через @IBAction

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    print("Editing ended. Final text: \(sender.text ?? "")")
}
```

- В Interface Builder привязывается к событию **Editing Did End**
    

### Через addTarget

```swift
let textField = UITextField()
textField.addTarget(self, action: #selector(textFieldEditingDidEnd(_:)), for: .editingDidEnd)

@objc func textFieldEditingDidEnd(_ sender: UITextField) {
    print("Editing ended programmatically: \(sender.text ?? "")")
}
```

- Программный способ отслеживания завершения редактирования
    

---

## 4. Примеры от простого к сложному

### Пример 1. Логирование финального текста

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    print("Final text: \(sender.text ?? "")")
}
```

- Просто выводим текст после завершения ввода
    

---

### Пример 2. Валидация текста после ввода

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    if let text = sender.text, text.count < 3 {
        print("Text too short")
    } else {
        print("Text is valid")
    }
}
```

- Проверяем текст **после окончания редактирования**
    

---

### Пример 3. Скрытие клавиатуры при завершении

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    sender.resignFirstResponder()
}
```

- Убираем клавиатуру, когда пользователь закончил ввод
    

---

### Пример 4. Автообновление интерфейса

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    label.text = sender.text
}
@IBOutlet weak var label: UILabel!
```

- Обновляем другой элемент интерфейса после ввода текста
    

---

### Пример 5. Сохранение данных

```swift
@IBAction func textFieldEditingDidEnd(_ sender: UITextField) {
    if let text = sender.text {
        UserDefaults.standard.set(text, forKey: "username")
    }
}
```

- Сохраняем введённое значение после завершения редактирования
    

---

## 5. Особенности EditingDidEnd

1. Срабатывает **только один раз после завершения редактирования**
    
2. Отличается от **[[EditingChanged]]**, которое срабатывает при каждом вводе
    
3. Используется для:
    
    - Валидации текста
        
    - Обновления UI
        
    - Сохранения данных
        
4. Поддерживается как **@IBAction**, так и **addTarget**
    

---

## 6. Итог

- **EditingDidEnd** = событие завершения редактирования UITextField
    
- Позволяет:
    
    - Получить финальный текст после ввода
        
    - Сделать валидацию и обработку данных
        
    - Управлять UI и клавиатурой
        
- Поддерживается **Interface Builder** и программно через **addTarget**
    

---
