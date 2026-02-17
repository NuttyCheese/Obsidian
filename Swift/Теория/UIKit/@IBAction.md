**@IBAction** — это специальный атрибут в [[Swift]], который делает метод **видимым и доступным** для **Interface Builder** (IB) в [[Xcode]].

Он позволяет **привязывать** методы к событиям UI-элементов (кнопки, переключатели, слайдеры, жесты и т.д.) прямо в визуальном редакторе — без написания кода `addTarget(_:action:)` вручную.

### Главное, для чего нужен @IBAction (2026 актуально)

- Связывает **UI-событие** (нажатие кнопки, изменение слайдера, выбор сегмента и т.д.) с методом в коде
- Работает **только** с элементами, которые наследуются от [[UIControl]] или поддерживают target-action ([[UIButton]], [[UISwitch]], [[UISlider]], [[UISegmentedControl]], [[UITextField]], [[UIBarButtonItem]] и т.д.)
- Под капотом автоматически добавляет **`@objc`** — метод становится видимым для [[Objective-C]] [[Runtime]]
- Позволяет **drag-and-drop** подключение в Interface Builder (Ctrl+Drag от элемента к коду)

### Ключевые правила написания @IBAction

```swift
@IBAction func имяМетода(_ sender: ТипЭлемента) {
    // код
}
```

- **@IBAction** — обязательно перед func
- **sender** — параметр, через который приходит объект, вызвавший действие (обычно UIButton, UISwitch и т.д.)
- Тип sender можно указывать конкретно ([[UIButton]]) или общо ([[any]], [[UIControl]])
- Метод может быть **private**, но тогда подключение в IB нужно делать вручную

### Самые частые примеры (2026 стиль)

#### 1. Кнопка (самый популярный случай)

```swift
@IBAction func loginButtonTapped(_ sender: UIButton) {
    print("Кнопка Войти нажата")
    // Логика входа
}
```

#### 2. Переключатель (UISwitch)

```swift
@IBAction func darkModeSwitchChanged(_ sender: UISwitch) {
    let isOn = sender.isOn
    overrideUserInterfaceStyle = isOn ? .dark : .light
}
```

#### 3. Слайдер (UISlider)

```swift
@IBAction func volumeSliderChanged(_ sender: UISlider) {
    let value = sender.value  // от 0.0 до 1.0
    player?.volume = value
}
```

#### 4. Сегментированный контрол (UISegmentedControl)

```swift
@IBAction func segmentChanged(_ sender: UISegmentedControl) {
    let index = sender.selectedSegmentIndex
    switch index {
    case 0: showGridView()
    case 1: showListView()
    default: break
    }
}
```

#### 5. Один метод на несколько элементов (общий sender)

```swift
@IBAction func controlValueChanged(_ sender: UIControl) {
    switch sender {
    case let button as UIButton:
        print("Нажата кнопка с тегом \(button.tag)")
    case let switchControl as UISwitch:
        print("Переключатель: \(switchControl.isOn ? "Вкл" : "Выкл")")
    case let slider as UISlider:
        print("Слайдер: \(slider.value)")
    default:
        break
    }
}
```

### Важные нюансы 2026 года

- **@IBAction** работает **только** с Interface Builder  
  Если ты добавляешь target-action в коде — пиши просто `@objc func`, без @IBAction
- **sender** можно опустить (редко):

```swift
@IBAction func simpleTap() {
    print("Тапнули")
}
```

- **Несколько событий на один метод** — подключай несколько раз в IB к разным событиям (Touch Up Inside, Value Changed и т.д.)
- **Swift 6 strict concurrency** — @IBAction-методы вызываются на главном потоке → безопасны  
  Но если внутри вызываешь async код — оборачивай в Task { await ... }
- **Документируйте** — пиши комментарий «@IBAction — обработка нажатия кнопки Войти»

**Короткий девиз 2026**:
> @IBAction — это когда ты хочешь **соединить кнопку/переключатель/слайдер с методом в коде** прямо в Interface Builder.  
> В 2026 году это **самый удобный** способ привязки UI-событий в UIKit-проектах со Storyboard/XIB.  
> В чистом коде (без IB) — используй просто @objc + addTarget.
