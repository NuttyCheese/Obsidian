**`UITextViewDelegate`** — это **протокол**, который предоставляет методы для реагирования на события **[[UITextView]]**:

- Начало редактирования (`textViewDidBeginEditing`)
    
- Окончание редактирования (`textViewDidEndEditing`)
    
- Изменение текста (`textViewDidChange`)
    
- Управление выбором текста (`textViewDidChangeSelection`)
    
- Обработка нажатий на ссылки
    

Используется через **свойство `delegate` текстового поля**.

> Проще говоря: `UITextViewDelegate` = «обработчик событий UITextView».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**delegate**|Свойство UITextView, куда присваивается объект, реализующий протокол|
|**textViewDidBeginEditing**|Вызывается после начала редактирования текста|
|**textViewDidEndEditing**|Вызывается после завершения редактирования текста|
|**textViewDidChange**|Вызывается при каждом изменении текста|
|**textViewDidChangeSelection**|Вызывается при изменении выделения текста|
|**shouldChangeTextIn**|Позволяет контролировать, какие изменения текста допустимы|
|**isEditable**|Можно ли редактировать текст (UITextView)|
|**isSelectable**|Можно ли выделять текст|

---

## 3. Основной синтаксис

### Назначение делегата

```swift
let textView = UITextView()
textView.delegate = self // self должен соответствовать UITextViewDelegate
```

### Реализация протокола

```swift
class ViewController: UIViewController, UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        print("Текст изменился: \(textView.text ?? "")")
    }
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Логирование текста при изменении

```swift
func textViewDidChange(_ textView: UITextView) {
    print("Текущий текст: \(textView.text ?? "")")
}
```

- Реагируем на любое изменение текста
    

---

### Пример 2. Скрытие клавиатуры при окончании редактирования

```swift
func textViewDidEndEditing(_ textView: UITextView) {
    textView.resignFirstResponder()
}
```

- Клавиатура скрывается, когда пользователь закончил ввод
    

---

### Пример 3. Ограничение количества символов

```swift
func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
    let maxLength = 200
    let currentText = textView.text ?? ""
    guard let stringRange = Range(range, in: currentText) else { return false }
    let updatedText = currentText.replacingCharacters(in: stringRange, with: text)
    return updatedText.count <= maxLength
}
```

- Пользователь не сможет ввести больше 200 символов
    

---

### Пример 4. Реализация placeholder

```swift
let placeholderLabel = UILabel()
placeholderLabel.text = "Введите комментарий..."
placeholderLabel.font = UIFont.systemFont(ofSize: 16)
placeholderLabel.textColor = .lightGray
placeholderLabel.frame = CGRect(x: 5, y: 8, width: textView.frame.width, height: 20)
textView.addSubview(placeholderLabel)

func textViewDidChange(_ textView: UITextView) {
    placeholderLabel.isHidden = !textView.text.isEmpty
}
```

- Имитация placeholder для UITextView
    

---

### Пример 5. Динамическая проверка текста

```swift
func textViewDidChangeSelection(_ textView: UITextView) {
    print("Выделение текста изменилось")
}
```

- Метод реагирует на изменение позиции курсора или выделения текста
    

---

## 5. Особенности UITextViewDelegate

1. Делегат должен быть **слабой ссылкой (`weak`)**, чтобы избежать retain cycle
    
2. Методы протокола **не обязательны**, их можно реализовать выборочно
    
3. Позволяет:
    
    - Реагировать на редактирование текста
        
    - Ограничивать длину текста или набор символов
        
    - Реализовать placeholder
        
    - Отслеживать выделение текста
        
4. Часто используется для: заметок, комментариев, сообщений, чатов
    

---

## 6. Итог

- **UITextViewDelegate** = протокол для обработки событий UITextView
    
- Позволяет:
    
    - Отслеживать начало и конец редактирования
        
    - Ограничивать ввод текста
        
    - Реагировать на изменения выделения
        
- Работает через свойство `delegate` текстового поля
    

---
