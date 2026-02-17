**`ValueChanged`** — это одно из событий (`UIControl.Event`) для элементов управления ([[UIControl]]), которое срабатывает, когда **значение элемента изменилось**.

Используется для:

- [[UISlider]] — ползунок
    
- [[UISwitch]] — переключатель
    
- [[UITextField]] с редактированием
    
- [[UIDatePicker]] — выбор даты/времени
    

---

## 2. События UIControl

`UIControl.Event.valueChanged` сигнализирует о том, что значение контроллера **изменилось пользователем**.  
Пример событий для сравнения:

| Событие            | Когда срабатывает                           |
| ------------------ | ------------------------------------------- |
| [[TouchUpInside]]  | Пользователь отпустил палец на кнопке       |
| [[EditingChanged]] | Текстовое поле изменило текст               |
| valueChanged       | Слайдер, свитч, датапикер изменили значение |

---

## 3. Пример с UISwitch

```swift
let toggle = UISwitch()
toggle.addTarget(self, action: #selector(switchChanged(_:)), for: .valueChanged)

@objc func switchChanged(_ sender: UISwitch) {
    if sender.isOn {
        print("Свитч включён")
    } else {
        print("Свитч выключен")
    }
}
```

---

## 4. Пример с UISlider

```swift
let slider = UISlider()
slider.minimumValue = 0
slider.maximumValue = 100
slider.addTarget(self, action: #selector(sliderChanged(_:)), for: .valueChanged)

@objc func sliderChanged(_ sender: UISlider) {
    print("Новое значение слайдера: \(sender.value)")
}
```

---

## 5. Особенности

- `.valueChanged` **срабатывает сразу**, как только пользователь изменяет значение.
    
- Для текстовых полей чаще используют `.editingChanged`, а `.valueChanged` применяют к `UISlider`, `UISwitch` и `UIDatePicker`.
    
- Можно использовать один и тот же метод для разных элементов, проверяя тип `sender`.
    

---

## 6. Итог

- **`ValueChanged`** = событие изменения значения элемента управления.
    
- Основные применяемые элементы: слайдер, свитч, дата-пикер.
    
- Используется через `addTarget(_:action:for:)`.
    

---
