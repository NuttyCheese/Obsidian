**`UILabel`** — это подкласс **[[Swift/Теория/UIKit/UIView]]**, предназначенный для отображения **текста** в приложении [[iOS]].

- Не интерактивен по умолчанию
    
- Позволяет настраивать:
    
    - Шрифт (`font`)
        
    - Цвет текста (`textColor`)
        
    - Выравнивание (`textAlignment`)
        
    - Количество строк (`numberOfLines`)
        
- Поддерживает **атрибутированный текст ([[NSAttributedString]])** для кастомизации стиля
    

> Проще говоря: `UILabel` = «виджет для отображения текста на экране».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**text**|Текст, который отображается в UILabel|
|**textColor**|Цвет текста|
|**font**|Шрифт и размер текста|
|**textAlignment**|Выравнивание текста (`.left`, `.center`, `.right`, `.justified`)|
|**numberOfLines**|Количество строк (0 = неограниченно)|
|**lineBreakMode**|Как обрезается текст, если не помещается (`.byTruncatingTail`, `.byWordWrapping`)|
|**attributedText**|Атрибутированный текст для цветного, жирного, подчеркнутого текста|
|**adjustsFontSizeToFitWidth**|Автоматически уменьшает размер шрифта, чтобы текст помещался в UILabel|

---

## 3. Основной синтаксис

### Через Interface Builder

```swift
@IBOutlet weak var label: UILabel!
label.text = "Hello, World!"
label.textColor = .systemBlue
label.font = UIFont.systemFont(ofSize: 18, weight: .medium)
label.textAlignment = .center
```

### Программно

```swift
let label = UILabel(frame: CGRect(x: 50, y: 50, width: 200, height: 50))
label.text = "Hello, World!"
label.textColor = .black
label.font = UIFont.systemFont(ofSize: 16)
label.textAlignment = .center
label.numberOfLines = 1
view.addSubview(label)
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простое текстовое отображение

```swift
label.text = "Привет, мир!"
label.textColor = .systemBlue
```

- Просто выводим текст на экран
    

---

### Пример 2. Многострочный текст

```swift
label.numberOfLines = 0
label.text = "Это очень длинный текст, который может занимать несколько строк в UILabel."
label.lineBreakMode = .byWordWrapping
```

- Текст автоматически переносится на несколько строк
    

---

### Пример 3. Атрибутированный текст

```swift
let attributedString = NSMutableAttributedString(string: "Hello, World!")
attributedString.addAttribute(.foregroundColor, value: UIColor.red, range: NSRange(location: 0, length: 5))
attributedString.addAttribute(.font, value: UIFont.boldSystemFont(ofSize: 20), range: NSRange(location: 7, length: 5))
label.attributedText = attributedString
```

- Позволяет делать текст разноцветным, жирным, подчеркнутым
    

---

### Пример 4. Автоматическое уменьшение размера текста

```swift
label.adjustsFontSizeToFitWidth = true
label.minimumScaleFactor = 0.5
label.text = "Очень длинный текст, который может не помещаться"
```

- Шрифт уменьшается, чтобы текст уместился в UILabel
    

---

### Пример 5. UILabel с анимацией изменения текста

```swift
UIView.transition(with: label, duration: 0.5, options: .transitionCrossDissolve) {
    label.text = "Новый текст с анимацией"
}
```

- Плавная смена текста с эффектом затухания
    

---

## 5. Особенности UILabel

1. **Не интерактивен по умолчанию**, используется только для отображения текста
    
2. Поддерживает **однострочный и многострочный текст**
    
3. Позволяет работать с **атрибутированным текстом** (`NSAttributedString`)
    
4. Можно комбинировать с **анимациями UIView**
    
5. Для интерактивности можно добавлять **[[UITapGestureRecognizer]]**
    

---

## 6. Итог

- **UILabel** = компонент для отображения текста
    
- Позволяет:
    
    - Отображать простой и многострочный текст
        
    - Настраивать шрифт, цвет, выравнивание
        
    - Использовать атрибутированный текст для кастомизации
        
    - Анимировать изменения текста
        

---
