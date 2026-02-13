## 1. Что такое `TouchDown`

**`TouchDown`** — это событие [[UIControl]], которое срабатывает **сразу в момент, когда пользователь касается элемента (например, кнопки)**.

- Используется в [[UIButton]], [[UISlider]] и других наследниках **UIControl**
    
- Отличается от [[TouchUpInside]] и [[TouchUpOutside]], которые срабатывают при отпускании пальца
    
- Часто применяется для **визуальной реакции кнопки на нажатие**
    

> Проще говоря: `TouchDown` = «пользователь коснулся кнопки».

---

## 2. Основные термины

| Термин                        | Описание                                                               |
| ----------------------------- | ---------------------------------------------------------------------- |
| **UIControl.Event**           | Перечисление событий, которые могут происходить с элементами UIControl |
| **TouchDown**                 | Событие при начале нажатия на элемент                                  |
| **Target-Action**             | Механизм связывания события и метода                                   |
| **[[@IBAction]] / addTarget** | Способы обработать событие в коде                                      |

---

## 3. Основной синтаксис

### Через @IBAction

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    print("Touch down!")
}
```

- В Interface Builder привязывается к событию **Touch Down**
    

### Через addTarget

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonTouchDown(_:)), for: .touchDown)

@objc func buttonTouchDown(_ sender: UIButton) {
    print("Touch down programmatically!")
}
```

- Программный способ отслеживания события
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое логирование

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    print("User started touching the button")
}
```

- Просто вывод в консоль
    

---

### Пример 2. Изменение цвета кнопки при нажатии

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    sender.backgroundColor = .gray
}
```

- Визуальная обратная связь при начале нажатия
    

---

### Пример 3. Возврат цвета при отпускании

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    sender.backgroundColor = .gray
}

@IBAction func buttonTouchUpInside(_ sender: UIButton) {
    sender.backgroundColor = .systemBlue
}
@IBAction func buttonTouchUpOutside(_ sender: UIButton) {
    sender.backgroundColor = .systemBlue
}
```

- Контролируем внешний вид кнопки на разных событиях
    

---

### Пример 4. addTarget с TouchDown

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonTouchDown(_:)), for: .touchDown)

@objc func buttonTouchDown(_ sender: UIButton) {
    print("TouchDown event triggered!")
}
```

- Программная обработка начала нажатия
    

---

### Пример 5. TouchDown с анимацией

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    UIView.animate(withDuration: 0.1) {
        sender.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
    }
}

@IBAction func buttonTouchUpInside(_ sender: UIButton) {
    UIView.animate(withDuration: 0.1) {
        sender.transform = .identity
    }
}
```

- Кнопка **сжимается при касании**, а возвращается в исходное состояние при отпускании
    

---

## 5. Особенности TouchDown

1. Срабатывает **сразу при касании**, до отпускания пальца
    
2. Отличается от `TouchUpInside` и `TouchUpOutside`
    
3. Часто используется для **визуальной обратной связи (эффект нажатия)**
    
4. Поддерживается **@IBAction** и **addTarget**
    
5. Можно комбинировать с другими событиями для полного UX
    

---

## 6. Итог

- **TouchDown** = событие UIControl при начале нажатия
    
- Позволяет:
    
    - Реализовать визуальные эффекты нажатия
        
    - Запускать предварительные действия до отпускания пальца
        
    - Использовать вместе с TouchUpInside / TouchUpOutside для контроля UX
        
- Поддерживается **Interface Builder** и программно через **addTarget**
    

---
