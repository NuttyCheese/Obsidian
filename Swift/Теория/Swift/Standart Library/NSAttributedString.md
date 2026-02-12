## 1. Что такое `NSAttributedString`

**`NSAttributedString`** — это **неизменяемый** объект текста с набором **атрибутов**, которые определяют, как текст отображается.

- Атрибуты могут быть:
    
    - Цвет (`foregroundColor`)
        
    - Шрифт (`font`)
        
    - Подчеркивание (`underlineStyle`)
        
    - Межстрочный интервал, выравнивание и др.
        
- Есть **изменяемый вариант `NSMutableAttributedString`**, позволяющий изменять атрибуты после создания
    
- Используется с **[[UILabel]], [[UITextView]], [[UIButton]]** для отображения форматированного текста
    

> Проще говоря: `NSAttributedString` = «текст + стиль».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Attributes**|Словарь `[NSAttributedString.Key: Any]` с ключами для стиля текста|
|**foregroundColor**|Цвет текста|
|**font**|Шрифт текста|
|**underlineStyle**|Подчеркивание|
|**NSMutableAttributedString**|Изменяемая версия NSAttributedString|
|**attributedText**|Свойство UILabel, UITextView и UIButton для отображения форматированного текста|
|**NSAttributedString.Key**|Перечисление всех возможных ключей атрибутов|

---

## 3. Основной синтаксис

### Создание с атрибутами

```swift
let attributes: [NSAttributedString.Key: Any] = [
    .foregroundColor: UIColor.red,
    .font: UIFont.boldSystemFont(ofSize: 18)
]

let attributedString = NSAttributedString(string: "Привет, мир!", attributes: attributes)
```

### Присвоение в UILabel

```swift
label.attributedText = attributedString
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простой цвет и шрифт

```swift
let attrString = NSAttributedString(
    string: "Hello, World!",
    attributes: [
        .foregroundColor: UIColor.blue,
        .font: UIFont.systemFont(ofSize: 20)
    ]
)
label.attributedText = attrString
```

- Просто задаем цвет и размер шрифта
    

---

### Пример 2. Подчеркивание текста

```swift
let attrString = NSAttributedString(
    string: "Подчеркнутый текст",
    attributes: [
        .underlineStyle: NSUnderlineStyle.single.rawValue,
        .foregroundColor: UIColor.black
    ]
)
label.attributedText = attrString
```

- Текст будет подчеркнут
    

---

### Пример 3. Разные атрибуты для частей текста

```swift
let mutableAttr = NSMutableAttributedString(string: "Hello, World!")
mutableAttr.addAttribute(.foregroundColor, value: UIColor.red, range: NSRange(location: 0, length: 5))
mutableAttr.addAttribute(.foregroundColor, value: UIColor.green, range: NSRange(location: 7, length: 5))
label.attributedText = mutableAttr
```

- Первые 5 символов красные, следующие 5 — зелёные
    

---

### Пример 4. Жирный и курсивный текст

```swift
let mutableAttr = NSMutableAttributedString(string: "Swift is awesome")
mutableAttr.addAttribute(.font, value: UIFont.boldSystemFont(ofSize: 18), range: NSRange(location: 0, length: 5))
mutableAttr.addAttribute(.font, value: UIFont.italicSystemFont(ofSize: 18), range: NSRange(location: 8, length: 7))
label.attributedText = mutableAttr
```

- Смешиваем разные стили шрифта в одном тексте
    

---

### Пример 5. Смешанные атрибуты: цвет, шрифт, подчеркивание

```swift
let mutableAttr = NSMutableAttributedString(string: "Swift ❤️ iOS")
mutableAttr.addAttributes([
    .foregroundColor: UIColor.systemRed,
    .font: UIFont.systemFont(ofSize: 20),
    .underlineStyle: NSUnderlineStyle.single.rawValue
], range: NSRange(location: 6, length: 1))
label.attributedText = mutableAttr
```

- Символ ❤️ будет красным, жирным и подчеркнутым
    

---

## 5. Особенности NSAttributedString

1. **Не изменяемый** (`NSAttributedString`) vs **изменяемый** (`NSMutableAttributedString`)
    
2. Атрибуты задаются через **словарь `[NSAttributedString.Key: Any]`**
    
3. Поддерживает все UI элементы, которые могут отображать текст
    
4. Можно комбинировать разные атрибуты для разных диапазонов текста
    
5. Полезен для создания **разноцветного текста, подчеркиваний, шрифтов и emoji**
    

---

## 6. Итог

- **NSAttributedString** = текст + атрибуты (цвет, шрифт, стиль)
    
- Используется для:
    
    - UILabel, UITextView, UIButton
        
    - Форматирования частей текста по-разному
        
    - Подчеркивания, изменения шрифта, цвета, стиля
        

---
