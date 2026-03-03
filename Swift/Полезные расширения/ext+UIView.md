#extension #uiview 

---

```swift
extension UIView {
    /// Включает пользовательское взаимодействие
    func enableUserInteraction() {
        isUserInteractionEnabled = true
    }
}
```

### Что делает этот extension?

Это очень простое и очень популярное расширение, которое добавляет к любому [[UIView]] (и всем его наследникам: [[UIButton]], [[UILabel]], [[UIImageView]], [[UIStackView]], [[UITableViewCell]] и т.д.) удобный метод:

```swift
view.enableUserInteraction()
```

который просто делает:

```swift
view.isUserInteractionEnabled = true
```

### Зачем вообще такое писать?

1. **Читаемость кода сильно улучшается**  
   Сравни:

   ```swift
   button.isUserInteractionEnabled = true
   button.isUserInteractionEnabled = false
   
   // vs
   
   button.enableUserInteraction()
   button.disableUserInteraction()   // если добавишь такую же функцию
   ```

   Второй вариант читается как обычный английский язык → меньше когнитивной нагрузки.

2. **Цепочки (method chaining)**  
   Очень удобно, когда настраиваешь view в одну цепочку:

   ```swift
   let label = UILabel()
       .withText("Загрузка...")
       .withTextColor(.gray)
       .withFont(.systemFont(ofSize: 15))
       .enableUserInteraction()           // ← вот здесь красиво ложится
       .withGesture(UITapGestureRecognizer(target: self, action: #selector(tappedLabel)))
   ```

3. **Симметрия с disableUserInteraction()**  
   Обычно такое расширение идёт парой:

   ```swift
   extension UIView {
       func enableUserInteraction() {
           isUserInteractionEnabled = true
       }
       
       func disableUserInteraction() {
           isUserInteractionEnabled = false
       }
   }
   ```

   Тогда код выглядит очень симметрично и логично:

   ```swift
   containerView.disableUserInteraction()           // пока идёт анимация / загрузка
   // ... через 2 секунды
   containerView.enableUserInteraction()
   ```

### Реальные ситуации, где это часто используют

- Отключают взаимодействие на весь экран во время загрузки / анимации / пока показан [[UIAlertController]]
- Делают "замороженными" отдельные карточки/кнопки в коллекции во время редактирования
- Временно отключают взаимодействие при свайп-бэк жесте или при открытом боковом меню
- В тестах / UI-автотестах часто принудительно включают/выключают взаимодействие

### Пример использования (полный)

```swift
extension UIView {
    func enableUserInteraction() {
        isUserInteractionEnabled = true
    }
    
    func disableUserInteraction() {
        isUserInteractionEnabled = false
    }
}

// ────────────────────────────────────────────────

class ProfileViewController: UIViewController {
    
    @IBOutlet private weak var saveButton: UIButton!
    @IBOutlet private weak var overlayView: UIView!
    
    func startSavingAnimation() {
        // отключаем всё взаимодействие
        view.disableUserInteraction()
        saveButton.disableUserInteraction()
        
        overlayView.isHidden = false
        // запускаем спиннер и т.д.
    }
    
    func finishSaving() {
        // возвращаем взаимодействие
        view.enableUserInteraction()
        saveButton.enableUserInteraction()
        
        overlayView.isHidden = true
    }
}
```

### Возможные улучшения (часто добавляют)

```swift
extension UIView {
    @discardableResult
    func enableUserInteraction() -> Self {
        isUserInteractionEnabled = true
        return self
    }
    
    @discardableResult
    func disableUserInteraction() -> Self {
        isUserInteractionEnabled = false
        return self
    }
}
```

Тогда можно писать цепочки ещё красивее:

```swift
imageView
    .enableUserInteraction()
    .addGesture(tapGesture)
```

---
---
---

```swift
extension UIView {
    
    /// Отключает autoresizing mask для Auto Layout
    func disableAutoresizingMask() {
        translatesAutoresizingMaskIntoConstraints = false
    }
    
    /// Добавляет несколько subviews
    func addSubviews(_ views: UIView...) {
        views.forEach { addSubview($0) }
    }
    
    /// Добавляет несколько subviews из массива
    func addSubviews(_ views: [UIView]) {
        views.forEach { addSubview($0) }
    }
}
```

### 1. `disableAutoresizingMask()`

**Что делает**  
Устанавливает свойство `translatesAutoresizingMaskIntoConstraints = false`

**Зачем это нужно**  
Когда вы создаёте view программно и собираетесь использовать **Auto Layout** (constraints), почти всегда нужно отключить автоматический перевод autoresizing mask в constraints.  
По умолчанию это свойство = `true` → система пытается создать для вас скрытые constraints из старой системы springs & struts → почти всегда это ломает ваш layout.

После вызова `disableAutoresizingMask()` (или прямой записи `false`) view становится «чистой» для ваших собственных constraints.

**Типичные варианты написания (самые популярные)**

```swift
func disableAutoresizingMask() {
    translatesAutoresizingMaskIntoConstraints = false
}

// или с возвратом self (для chaining)
@discardableResult
func disableAutoresizingMask() -> Self {
    translatesAutoresizingMaskIntoConstraints = false
    return self
}
```

**Реальный пример использования**

```swift
let container = UIView()
    .disableAutoresizingMask()           // ← почти всегда первым делом

let titleLabel = UILabel()
    .disableAutoresizingMask()
    .withText("Привет")
    .withTextColor(.label)

container.addSubview(titleLabel)

NSLayoutConstraint.activate([
    titleLabel.topAnchor.constraint(equalTo: container.topAnchor, constant: 16),
    titleLabel.leadingAnchor.constraint(equalTo: container.leadingAnchor, constant: 16),
    titleLabel.trailingAnchor.constraint(lessThanOrEqualTo: container.trailingAnchor, constant: -16)
])
```

### 2 & 3. `addSubviews(…)` — две перегрузки

**Что делают**

- `addSubviews(_ views: UIView...)` — variadic (можно передать сколько угодно view через запятую)
- `addSubviews(_ views: [UIView])` — принимает готовый массив

Обе просто вызывают `addSubview` в цикле.

**Зачем это удобно**

Убирает boilerplate-код:

```swift
// было (многострочно и некрасиво)
stack.addSubview(label)
stack.addSubview(button)
stack.addSubview(imageView)
stack.addSubview(spinner)

// стало (одна строка)
stack.addSubviews(label, button, imageView, spinner)

// или если уже есть массив
let children = [label, button, imageView, spinner]
stack.addSubviews(children)
```

**Самые популярные улучшения (часто встречаются вместе)**

```swift
extension UIView {
    
    @discardableResult
    func addSubviews(_ views: UIView...) -> Self {
        views.forEach { addSubview($0) }
        return self
    }
    
    @discardableResult
    func addSubviews(_ views: [UIView]) -> Self {
        views.forEach { addSubview($0) }
        return self
    }
    
    // бонус — часто добавляют ещё и disableAutoresizingMask сразу
    @discardableResult
    func addSubviewsAndDisableAutoresizingMask(_ views: UIView...) -> Self {
        views.forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
        return self
    }
}
```

**Реальный пример (очень частый паттерн 2024–2026)**

```swift
private func setupUI() {
    let stack = UIStackView()
        .disableAutoresizingMask()
        .withAxis(.vertical)
        .withSpacing(12)
        .withAlignment(.fill)
    
    let title = UILabel().withText("Настройки").disableAutoresizingMask()
    let subtitle = UILabel().withText("Версия 3.2.1").disableAutoresizingMask()
    let saveButton = UIButton(type: .system).withTitle("Сохранить").disableAutoresizingMask()
    
    stack.addSubviews(title, subtitle, saveButton)
    
    view.addSubview(stack)
    
    NSLayoutConstraint.activate([
        stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 24),
        stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
        stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
    ])
}
```

**Коротко — когда использовать каждую версию**

| Метод                                   | Когда удобно                                                                  |
| --------------------------------------- | ----------------------------------------------------------------------------- |
| `addSubviews(label, button)`            | 2–6 элементов прямо в коде, читается очень естественно                        |
| `addSubviews(array)`                    | элементы уже собраны в массив ([[map]], [[filter]], из другого метода и т.д.) |
| `addSubviewsAndDisableAutoresizingMask` | самый частый сценарий при программном создании UI                             |
