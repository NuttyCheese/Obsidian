## 1. Что такое `TouchUpOutside`

**`TouchUpOutside`** — это событие [[UIControl]], которое срабатывает **когда пользователь нажал на элемент (например, кнопку) и отпустил палец за пределами его границ**.

- Используется в [[UIButton]], [[UISlider]] и других наследниках **UIControl**
    
- Отличается от [[TouchUpInside]]: событие срабатывает **только если отпуск происходит вне границ**
    
- Часто применяется для **отмены действия или сброса состояния кнопки**
    

> Проще говоря: `TouchUpOutside` = «пользователь нажал на кнопку, но отпустил палец вне её границ».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIControl.Event**|Перечисление событий, которые могут происходить с элементами UIControl|
|**TouchUpOutside**|Событие при отпускании пальца за пределами кнопки|
|**Target-Action**|Механизм связывания события и метода|
|**@IBAction / addTarget**|Способы обработать событие в коде|

---

## 3. Основной синтаксис

### Через [[@IBAction]]

```swift
@IBAction func buttonTouchUpOutside(_ sender: UIButton) {
    print("Touch up outside!")
}
```

- В Interface Builder привязывается к событию **Touch Up Outside**
    

### Через addTarget

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonTouchUpOutside(_:)), for: .touchUpOutside)

@objc func buttonTouchUpOutside(_ sender: UIButton) {
    print("Touch up outside programmatically!")
}
```

- Программный способ отслеживания события
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое логирование

```swift
@IBAction func buttonTouchUpOutside(_ sender: UIButton) {
    print("User released touch outside the button")
}
```

- Просто вывод в консоль
    

---

### Пример 2. Отмена действия

```swift
@IBAction func cancelButtonAction(_ sender: UIButton) {
    actionCancelled = true
    print("Action cancelled")
}
var actionCancelled = false
```

- Используется для **отмены ранее запущенного действия**
    

---

### Пример 3. Сменить цвет кнопки

```swift
@IBAction func buttonTouchUpOutside(_ sender: UIButton) {
    sender.backgroundColor = .systemGray
}
```

- Изменение визуального состояния кнопки, если палец ушёл за границы
    

---

### Пример 4. Сравнение с TouchUpInside

```swift
@IBAction func buttonTouchDown(_ sender: UIButton) {
    sender.backgroundColor = .blue
}

@IBAction func buttonTouchUpInside(_ sender: UIButton) {
    sender.backgroundColor = .green
}

@IBAction func buttonTouchUpOutside(_ sender: UIButton) {
    sender.backgroundColor = .red
}
```

- Разные цвета для разных событий: нажатие, отпуск внутри и отпуск снаружи
    

---

### Пример 5. addTarget с несколькими событиями

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonEvent(_:)), for: [.touchUpInside, .touchUpOutside])

@objc func buttonEvent(_ sender: UIButton) {
    print("Event: \(sender.title(for: .normal) ?? "")")
}
```

- Один метод может обрабатывать **несколько типов событий** одновременно
    

---

## 5. Особенности TouchUpOutside

1. Срабатывает **только при отпускании пальца вне границ**
    
2. Отличается от `TouchDown` (срабатывает при нажатии) и `TouchUpInside` (отпуск внутри)
    
3. Часто используется для **отмены действия или сброса состояния**
    
4. Поддерживается как **@IBAction**, так и **addTarget**
    
5. Полезно для создания **интерактивного UX**, где важно понимать, что пользователь «отменил» нажатие
    

---

## 6. Итог

- **TouchUpOutside** = событие UIControl при отпускании пальца за пределами элемента
    
- Позволяет:
    
    - Отслеживать отмену действия
        
    - Изменять внешний вид кнопки при уходе пальца
        
    - Использовать вместе с [[TouchDown]] и TouchUpInside для полного контроля UX
        
- Поддерживается **Interface Builder** и **код**
    

---
