## 1. Что такое `UIControl`

**`UIControl`** — это подкласс **[[Swift/Теория/UIKit/UIView]]**, предназначенный для обработки **событий пользовательского взаимодействия**.

- Все интерактивные элементы [[UIKit]] наследуются от [[UIControl]]:
    
    - **[[UIButton]]**
        
    - **[[UISlider]]**
        
    - **[[UISwitch]]**
        
    - **[[UITextField]]**
        
- С помощью **target-action** можно связывать события элемента с методами в коде
    
- Поддерживает множество событий: [[TouchDown]], [[TouchUpInside]], [[ValueChanged]], [[EditingChanged]] и др.
    

> Проще говоря: `UIControl` = «базовый интерактивный элемент, который умеет реагировать на действия пользователя».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIView**|Базовый класс для всех элементов интерфейса|
|**UIControl**|Наследник UIView для интерактивных элементов|
|**Target-Action**|Механизм связывания события с методом|
|**UIControl.Event**|Тип события: нажатие, отпуск, изменение значения|
|**addTarget / removeTarget**|Методы для программной привязки событий|
|**isEnabled / isSelected / isHighlighted**|Состояния интерактивного элемента|

---

## 3. Основной синтаксис

### Программное создание UIControl

```swift
let button = UIButton(type: .system) // UIButton наследуется от UIControl
button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    print("Button tapped!")
}
```

- Метод `addTarget(_:action:for:)` связывает событие с кодом
    
- Событие определяется через **UIControl.Event**
    

---

## 4. Примеры от простого к сложному

### Пример 1. UIButton с событием TouchUpInside

```swift
let button = UIButton(type: .system)
button.setTitle("Tap Me", for: .normal)
button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    print("Button tapped!")
}
```

- Используем target-action для обработки нажатия
    

---

### Пример 2. UISlider с ValueChanged

```swift
let slider = UISlider()
slider.minimumValue = 0
slider.maximumValue = 100
slider.addTarget(self, action: #selector(sliderChanged(_:)), for: .valueChanged)

@objc func sliderChanged(_ sender: UISlider) {
    print("Slider value: \(sender.value)")
}
```

- Обработка изменения значения ползунка
    

---

### Пример 3. UISwitch с ValueChanged

```swift
let toggle = UISwitch()
toggle.addTarget(self, action: #selector(switchChanged(_:)), for: .valueChanged)

@objc func switchChanged(_ sender: UISwitch) {
    print("Switch is \(sender.isOn ? "ON" : "OFF")")
}
```

- Реакция на переключение состояния
    

---

### Пример 4. UIButton с несколькими событиями

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonEvent(_:)), for: [.touchDown, .touchUpInside, .touchUpOutside])

@objc func buttonEvent(_ sender: UIButton) {
    print("Event triggered for button!")
}
```

- Один метод обрабатывает **несколько событий** одновременно
    

---

### Пример 5. UIControl кастомный

```swift
class CustomControl: UIControl {
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .systemBlue
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        sendActions(for: .touchDown)
        backgroundColor = .gray
    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesEnded(touches, with: event)
        sendActions(for: .touchUpInside)
        backgroundColor = .systemBlue
    }
}
```

- Создаём **свой кастомный UIControl**, который генерирует события вручную
    

---

## 5. Особенности UIControl

1. **Подкласс UIView**, поэтому можно размещать на экране и изменять frame
    
2. Поддерживает **target-action** для любого интерактивного действия
    
3. Поддерживает **состояния**: `isEnabled`, `isSelected`, `isHighlighted`
    
4. Можно использовать для **кнопок, слайдеров, переключателей и кастомных контролов**
    
5. Позволяет создавать **сложные интерактивные элементы с кастомной логикой**
    

---

## 6. Итог

- **UIControl** = базовый интерактивный элемент в UIKit
    
- Позволяет:
    
    - Обрабатывать нажатия, изменения значений, касания
        
    - Использовать target-action или @IBAction
        
    - Создавать кастомные интерактивные элементы
        
- Наследуется всеми стандартными интерактивными элементами: **UIButton, UISlider, UISwitch, UITextField**
    

---
