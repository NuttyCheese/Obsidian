**`UITextView`** — это **многострочное текстовое поле**, которое позволяет пользователю:

- Вводить и редактировать текст
    
- Отображать текст с форматированием (атрибуты, ссылки)
    
- Скроллить текст, если он превышает размеры поля
    

Особенности:

- Поддерживает **делегаты ([[UITextViewDelegate]])**
    
- Может быть **редактируемым** (`isEditable`) или **только для чтения** (`isSelectable`)
    
- Можно использовать для:
    
    - Заметок
        
    - Чат-полей
        
    - Описаний и комментариев
        

> Проще говоря: `UITextView` = «многострочное поле для ввода и отображения текста».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**text**|Текущий текст|
|**attributedText**|Текст с атрибутами (цвет, шрифт, подчеркивание)|
|**isEditable**|Можно ли редактировать текст|
|**isSelectable**|Можно ли выделять текст|
|**delegate**|Делегат, который реагирует на события (ввод, выделение, изменение)|
|**UITextViewDelegate**|Протокол для обработки событий начала/окончания редактирования, изменения текста и др.|
|**scrollEnabled**|Можно ли прокручивать текст|
|**keyboardType**|Тип клавиатуры (`.default`, `.numberPad`, `.emailAddress` и др.)|
|**textContainerInset**|Отступы текста внутри UITextView|

---

## 3. Основной синтаксис

### Создание UITextView программно

```swift
let textView = UITextView(frame: CGRect(x: 20, y: 200, width: 300, height: 150))
textView.text = "Введите текст..."
textView.font = UIFont.systemFont(ofSize: 16)
textView.textColor = .black
textView.isEditable = true
textView.isSelectable = true
textView.delegate = self
view.addSubview(textView)
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший UITextView

```swift
let textView = UITextView()
textView.text = "Привет, мир!"
view.addSubview(textView)
```

- Просто создаём поле для текста
    

---

### Пример 2. Настройка шрифта и цвета

```swift
textView.font = UIFont.systemFont(ofSize: 18)
textView.textColor = .systemBlue
```

- Меняем стиль текста
    

---

### Пример 3. Делегат для обработки изменений текста

```swift
class ViewController: UIViewController, UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        print("Текст изменился: \(textView.text ?? "")")
    }
}
```

- Делегат позволяет реагировать на каждое изменение текста
    

---

### Пример 4. UITextView с placeholder

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

- Имитируем placeholder, так как UITextView не имеет встроенного свойства `placeholder`
    

---

### Пример 5. UITextView с атрибутами

```swift
let attrString = NSMutableAttributedString(string: "Hello, Swift!")
attrString.addAttribute(.foregroundColor, value: UIColor.red, range: NSRange(location: 0, length: 5))
attrString.addAttribute(.font, value: UIFont.boldSystemFont(ofSize: 18), range: NSRange(location: 7, length: 5))
textView.attributedText = attrString
```

- Отображаем многострочный текст с разными стилями
    

---

## 5. Особенности UITextView

1. **Многострочный текст** (в отличие от [[UITextField]])
    
2. Поддерживает **прокрутку текста**
    
3. Делегат позволяет:
    
    - Реагировать на начало и конец редактирования
        
    - Отслеживать изменения текста
        
    - Обрабатывать выделение текста
        
4. Можно использовать **атрибуты для форматирования текста**
    
5. Часто используется для: заметок, комментариев, сообщений
    

---

## 6. Итог

- **UITextView** = многострочное текстовое поле для ввода и отображения текста
    
- Позволяет:
    
    - Настраивать шрифт, цвет, атрибуты текста
        
    - Отслеживать изменения через делегат
        
    - Скроллить длинный текст
        
- Отличие от UITextField: **UITextField — однострочный**, UITextView — многострочный
    

---
