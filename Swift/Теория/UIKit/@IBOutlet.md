## 1. Что такое `@IBOutlet`

**`@IBOutlet`** — это специальный атрибут [[Swift]], который позволяет **связывать переменные в коде с UI-элементами**, созданными в Interface Builder (Storyboard или [[xib]]).

- Используется только в **классах, наследующих `NSObject`**, чаще всего [[UIViewController]] или [[Swift/Теория/UIKit/UIView]]
    
- Позволяет **получить доступ к UI-элементам напрямую из кода**
    
- Атрибут **под капотом использует [[@objc]]**, чтобы runtime мог установить связь между интерфейсом и свойством
    

> Проще говоря: `@IBOutlet` = «свойство в коде, которое связано с элементом интерфейса в IB».

---

## 2. Основные термины

| Термин                     | Описание                                                                         |
| -------------------------- | -------------------------------------------------------------------------------- |
| **Interface Builder (IB)** | Визуальный редактор интерфейсов в Xcode                                          |
| **Outlet**                 | Связь между свойством кода и UI-элементом                                        |
| **UIControl / UIView**     | Типы элементов интерфейса, с которыми работает IBOutlet                          |
| **@objc**                  | Атрибут, который делает свойство доступным для [[Objective-C]] [[Runtime]]       |
| **[[Optional]]**           | Обычно IBOutlet делается Optional, так как связь может быть не установлена сразу |

---

## 3. Основной синтаксис

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var titleLabel: UILabel!
}
```

- `weak` — предотвращает **retain cycle**
    
- [[UILabel]]! — Implicitly Unwrapped Optional, чтобы можно было использовать до установки соединения
    
- После подключения в Interface Builder, `titleLabel` становится доступным в коде
    

---

## 4. Примеры от простого к сложному

### Пример 1. UILabel

```swift
@IBOutlet weak var label: UILabel!

override func viewDidLoad() {
    super.viewDidLoad()
    label.text = "Hello, World!"
}
```

- Присваиваем текст напрямую через IBOutlet
    

---

### Пример 2. [[UIButton]]

```swift
@IBOutlet weak var button: UIButton!

override func viewDidLoad() {
    super.viewDidLoad()
    button.setTitle("Tap me", for: .normal)
}
```

- Меняем **текст кнопки** через IBOutlet
    

---

### Пример 3. [[UIImageView]]

```swift
@IBOutlet weak var imageView: UIImageView!

override func viewDidLoad() {
    super.viewDidLoad()
    imageView.image = UIImage(named: "example")
}
```

- Загружаем картинку в ImageView через IBOutlet
    

---

### Пример 4. UISlider + IBOutlet

```swift
@IBOutlet weak var slider: UISlider!

override func viewDidLoad() {
    super.viewDidLoad()
    slider.value = 0.5
}
```

- Устанавливаем начальное значение ползунка через IBOutlet
    

---

### Пример 5. IBOutlet Collection

```swift
@IBOutlet var buttons: [UIButton]!

override func viewDidLoad() {
    super.viewDidLoad()
    for (index, button) in buttons.enumerated() {
        button.setTitle("Button \(index + 1)", for: .normal)
    }
}
```

- IBOutlet Collection = массив UI-элементов, которые можно обработать одновременно
    

---

## 5. Особенности @IBOutlet

1. Атрибут **связывает код и Interface Builder**
    
2. Обычно используется с **`weak`** и **implicitly unwrapped optional (!)**
    
3. Работает только с классами, наследующими **NSObject**
    
4. Можно создавать **IBOutlet Collection** для группы элементов
    
5. Позволяет **изменять свойства UI в коде** после загрузки интерфейса
    

---

## 6. Итог

- **@IBOutlet** = свойство для связи с UI-элементом из Interface Builder
    
- Позволяет:
    
    - UILabel, UIButton, UIImageView, UISlider и др. быть доступны в коде
        
    - IBOutlet Collection для группы элементов
        
    - Работает через Objective-C runtime и `weak` ссылку
        

---
