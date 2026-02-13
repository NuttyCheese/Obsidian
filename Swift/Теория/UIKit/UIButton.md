## 1. Что такое `UIButton`

**`UIButton`** — это подкласс **[[UIControl]]** в [[UIKit]], предназначенный для создания кнопок с интерактивными событиями.

- Может отображать: **текст, изображение, комбинацию текста и изображения**
    
- Реагирует на различные **события пользователя** ([[TouchUpInside]], [[TouchDown]] и др.)
    
- Может быть **создан в Interface Builder или программно**
    

> Проще говоря: `UIButton` = «кнопка, на которую можно нажать, чтобы вызвать действие».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIControl**|Базовый класс для интерактивных элементов (кнопки, слайдеры, переключатели)|
|**Target-Action**|Механизм связывания кнопки с методом, вызываемым при событии|
|**State**|Состояние кнопки: normal, highlighted, selected, disabled|
|**Title / Image**|Свойства кнопки для текста и изображения|
|**@IBAction / addTarget**|Способы обработать нажатие кнопки|

---

## 3. Основной синтаксис

### Через Interface Builder

```swift
@IBOutlet weak var myButton: UIButton!

@IBAction func buttonTapped(_ sender: UIButton) {
    print("Button tapped!")
}
```

- `IBOutlet` связывает кнопку с кодом
    
- `IBAction` реагирует на события, чаще всего **TouchUpInside**
    

### Программно

```swift
let button = UIButton(type: .system)
button.setTitle("Tap me", for: .normal)
button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    print("Button tapped programmatically!")
}
```

- Создание и настройка кнопки полностью через код
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простая кнопка с текстом

```swift
let button = UIButton(type: .system)
button.setTitle("Press Me", for: .normal)
```

- Создаём кнопку с системным стилем и текстом
    

---

### Пример 2. Кнопка с изображением

```swift
let button = UIButton(type: .custom)
button.setImage(UIImage(named: "icon"), for: .normal)
```

- Кнопка отображает картинку вместо текста
    

---

### Пример 3. Кнопка с изменением цвета при нажатии

```swift
let button = UIButton(type: .system)
button.setTitle("Click Me", for: .normal)
button.setTitleColor(.blue, for: .normal)
button.setTitleColor(.gray, for: .highlighted)
```

- Используем **разные цвета текста** для состояний кнопки
    

---

### Пример 4. Кнопка с cornerRadius и shadow

```swift
let button = UIButton(type: .system)
button.setTitle("Rounded", for: .normal)
button.backgroundColor = .systemBlue
button.layer.cornerRadius = 10
button.layer.shadowColor = UIColor.black.cgColor
button.layer.shadowOpacity = 0.3
button.layer.shadowOffset = CGSize(width: 0, height: 2)
```

- Настройка внешнего вида кнопки через **[[CALayer]]**
    

---

### Пример 5. Кнопка с target-action программно

```swift
let button = UIButton(type: .system)
button.setTitle("Tap Me", for: .normal)
button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    sender.setTitle("Tapped!", for: .normal)
}
```

- Программная обработка нажатия кнопки и изменение текста при тапе
    

---

## 5. Особенности UIButton

1. **Подкласс UIControl** → поддерживает target-action
    
2. Поддерживает **разные состояния**: `.normal`, `.highlighted`, `.disabled`, `.selected`
    
3. Может отображать **текст, изображение или оба**
    
4. Настраивается через **Interface Builder** или **код**
    
5. Поддерживает **анимации, cornerRadius, shadow** и другие визуальные эффекты
    

---

## 6. Итог

- **UIButton** = интерактивная кнопка в [[UIKit]]
    
- Позволяет:
    
    - Отображать текст и изображение
        
    - Обрабатывать события через **@IBAction / addTarget**
        
    - Настраивать внешний вид (цвет, тень, скругление)
        
    - Использовать разные состояния кнопки
        

---
