## 1. Что такое `@IBAction`

**`@IBAction`** — это специальный атрибут [[Swift]], который делает метод доступным для **Interface Builder**.

- Используется в **[[UIViewController]] или классах, наследующих NSObject**
    
- Позволяет привязывать метод к событиям UI, например:
    
    - кнопка нажата ([[TouchUpInside]])
        
    - переключатель изменён (`Value Changed`)
        
- На самом деле, под капотом метод помечается как **`@objc`**, чтобы его можно было вызвать через [[Objective-C]] [[Runtime]]
    

> Проще говоря: `@IBAction` = «метод, который Interface Builder может вызвать при взаимодействии пользователя с UI».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Interface Builder**|Визуальный редактор интерфейсов в Xcode|
|**Target-Action**|Механизм, который связывает UI-элемент с методом|
|**Selector**|Уникальный идентификатор метода (`#selector`)|
|**IBAction**|Атрибут Swift для связи метода с UI-событием|
|**@objc**|Атрибут, который делает метод доступным для Objective-C runtime|

---

## 3. Основной синтаксис

```swift
import UIKit

class ViewController: UIViewController {
    @IBAction func buttonTapped(_ sender: UIButton) {
        print("Button tapped!")
    }
}
```

- `_ sender: UIButton` — ссылка на элемент, вызвавший действие
    
- Метод можно подключить к кнопке в Interface Builder
    

---

## 4. Примеры от простого к сложному

### Пример 1. Кнопка с простым действием

```swift
@IBAction func buttonPressed(_ sender: UIButton) {
    print("Button pressed")
}
```

- При нажатии кнопки в UI вызовется этот метод
    

---

### Пример 2. [[UISwitch]]

```swift
@IBAction func switchChanged(_ sender: UISwitch) {
    if sender.isOn {
        print("Switch is ON")
    } else {
        print("Switch is OFF")
    }
}
```

- Используется для отслеживания **состояния переключателей**
    

---

### Пример 3. [[UISlider]]

```swift
@IBAction func sliderChanged(_ sender: UISlider) {
    print("Slider value: \(sender.value)")
}
```

- Позволяет получить **текущее значение ползунка**
    

---

### Пример 4. [[UIButton]] с анимацией

```swift
@IBAction func animateButton(_ sender: UIButton) {
    UIView.animate(withDuration: 0.5) {
        sender.alpha = 0.5
    }
}
```

- Можно добавить **анимацию или логику изменения UI**
    

---

### Пример 5. Использование с multiple sender types

```swift
@IBAction func controlChanged(_ sender: UIControl) {
    if let button = sender as? UIButton {
        print("Button tapped: \(button.tag)")
    } else if let slider = sender as? UISlider {
        print("Slider value: \(slider.value)")
    }
}
```

- Один метод может обрабатывать **несколько типов UI-элементов**
    

---

## 5. Особенности @IBAction

1. Метод должен быть **func**, обычно с параметром `sender`
    
2. Под капотом — это **`@objc`**
    
3. Используется только с **UI-элементами в Interface Builder**
    
4. Может быть **private**, но тогда подключение в IB может потребовать ручного соединения
    
5. Позволяет обрабатывать **события UI без прямого кода добавления target-action**
    

---

## 6. Итог

- **@IBAction** = метод для связывания с UI-событиями
    
- Позволяет:
    
    - Кнопкам (`UIButton`)
        
    - Переключателям (`UISwitch`)
        
    - Ползункам (`UISlider`)
        
    - Другим контролам реагировать на действия пользователя
        
- Работает через **Objective-C runtime** и селекторы
    
- Может быть read-only ([[func]]) и optional (через optional targets)
    

---
