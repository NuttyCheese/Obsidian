## 1. Что такое `TouchUpInside`

**`TouchUpInside`** — это событие [[UIControl]], которое срабатывает **когда пользователь нажал на элемент (например, кнопку) и отпустил палец внутри его границ**.

- Используется в [[UIButton]], [[UISlider]], [[UISwitch]] и других наследниках `UIControl`
    
- Часто связывается с методом через **[[@IBAction]]** или **addTarget**
    
- Наиболее распространённое событие для кнопок, так как **гарантирует, что нажатие произошло внутри кнопки**
    

> Проще говоря: `TouchUpInside` = «пользователь нажал и отпустил палец на кнопке».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIControl.Event**|Перечисление событий, которые могут происходить с элементами UIControl|
|**TouchUpInside**|Пользователь нажал и отпустил палец внутри границ элемента|
|**Target-Action**|Механизм связывания события и метода|
|**@IBAction / addTarget**|Способы обработать событие в коде|

---

## 3. Основной синтаксис

### Через @IBAction

```swift
@IBAction func buttonTapped(_ sender: UIButton) {
    print("Button tapped!")
}
```

- В Interface Builder связывается с событием **Touch Up Inside**
    

### Через addTarget

```swift
let button = UIButton(type: .system)
button.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    print("Button tapped via addTarget!")
}
```

- Программный способ, не используя Interface Builder
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простая кнопка через IBAction

```swift
@IBAction func buttonPressed(_ sender: UIButton) {
    print("Button pressed")
}
```

- Вызывается **только при TouchUpInside**
    

---

### Пример 2. Кнопка с анимацией

```swift
@IBAction func animateButton(_ sender: UIButton) {
    UIView.animate(withDuration: 0.3) {
        sender.alpha = 0.5
    }
}
```

- Реакция на нажатие с **анимацией изменения прозрачности**
    

---

### Пример 3. Кнопка с изменением текста

```swift
@IBAction func changeLabelText(_ sender: UIButton) {
    myLabel.text = "Button was tapped"
}
@IBOutlet weak var myLabel: UILabel!
```

- Обновление UI через **IBOutlet**
    

---

### Пример 4. Кнопка с таймером

```swift
@IBAction func startTimer(_ sender: UIButton) {
    Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
        print("Tick")
    }
}
```

- TouchUpInside запускает **таймер или процесс**
    

---

### Пример 5. addTarget с несколькими кнопками

```swift
let button1 = UIButton()
let button2 = UIButton()

button1.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)
button2.addTarget(self, action: #selector(buttonTapped(_:)), for: .touchUpInside)

@objc func buttonTapped(_ sender: UIButton) {
    print("Tapped button: \(sender.tag)")
}
```

- Один метод может обрабатывать **несколько кнопок** через `sender.tag`
    

---

## 5. Особенности TouchUpInside

1. Срабатывает **только при отпускании пальца внутри кнопки**
    
2. Отличается от `TouchDown` (срабатывает при нажатии) и [[TouchUpOutside]] (отпуск вне кнопки)
    
3. Наиболее безопасное событие для **кнопок и действий пользователя**
    
4. Можно использовать через **Interface Builder (@IBAction)** или **код (addTarget)**
    

---

## 6. Итог

- **TouchUpInside** = событие UIControl, когда палец нажал и отпустил кнопку внутри её границ
    
- Позволяет:
    
    - Вызывать методы через **IBAction**
        
    - Программно через **addTarget**
        
    - Обновлять UI, запускать таймеры, анимации, логику приложения
        
- Используется для **кнопок и интерактивных элементов**
    

---
