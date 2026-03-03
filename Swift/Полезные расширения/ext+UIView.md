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

---
---
---
Вот разбор этих трёх методов для закругления углов и добавления границы — это классика UIKit, которую до сих пор активно используют в 2025–2026 годах (хотя в новых проектах часто переходят на SwiftUI или более современные подходы).

```swift
extension UIView {
    
    /// Закругляет указанные углы
    func roundCorners(_ corners: UIRectCorner, radius: CGFloat) {
        if bounds == .zero {
            DispatchQueue.main.asyncAfter(deadline: .now()) { [weak self] in
                self?.roundCorners(corners, radius: radius)
            }
            return
        }
        
        let path = UIBezierPath(
            roundedRect: bounds,
            byRoundingCorners: corners,
            cornerRadii: CGSize(width: radius, height: radius)
        )
        
        let mask = CAShapeLayer()
        mask.path = path.cgPath
        mask.frame = bounds
        layer.mask = mask
    }
    
    /// Закругляет все углы
    func roundAllCorners(radius: CGFloat) {
        layer.cornerRadius = radius
        layer.masksToBounds = true
    }
    
    /// Добавляет границу
    func addBorder(color: UIColor, width: CGFloat) {
        layer.borderColor = color.cgColor
        layer.borderWidth = width
    }
}
```

### 1. `roundAllCorners(radius:)`

**Что делает**  
Самый простой и быстрый способ закруглить **все четыре** угла.

**Плюсы**  
- Очень производительный (не вызывает offscreen rendering в большинстве случаев до iOS 13–14)  
- Поддерживает тени (shadow) без дополнительных костылей, если `masksToBounds = false`  
- Работает с `clipsToBounds` (но тогда тень обрезается)

**Минусы**  
- Только все углы одновременно  
- С iOS 11+ можно выборочно закруглять через `maskedCorners`, но здесь используется старый стиль

**Когда использовать в 2025–2026**  
- Простые карточки, кнопки, аватарки  
- Когда нужна тень + закругление

```swift
cardView.roundAllCorners(radius: 16)
// или современный вариант (iOS 11+)
cardView.layer.cornerRadius = 16
cardView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMaxYCorner] // только верх
```

### 2. `roundCorners(_:radius:)`

**Что делает**  
Закругляет **только выбранные** углы через маску (`CAShapeLayer` + `UIBezierPath`).

**Особенности реализации**  
- Проверяет `bounds == .zero` и откладывает выполнение на следующий runloop (полезно при вызове в `init` или до `layoutSubviews`)  
- Это **единственный** надёжный способ до iOS 11 выборочно закруглять углы

**Плюсы**  
- Работает на всех версиях iOS  
- Можно закруглять любые комбинации углов

**Минусы**  
- Вызывает **offscreen rendering** → хуже производительность (особенно в скролле / коллекциях)  
- Сбрасывает тень (если была) — маска обрезает всё, включая shadow  
- Нужно обновлять маску при изменении размера view (в `layoutSubviews`)

**Типичные улучшения (что часто добавляют)**

```swift
override func layoutSubviews() {
    super.layoutSubviews()
    roundCorners([.topLeft, .topRight], radius: 20)  // перевызываем при ресайзе
}
```

Или делают метод с возвратом `Self` + обновление фрейма:

```swift
@discardableResult
func roundCorners(_ corners: UIRectCorner, radius: CGFloat) -> Self {
    // ... тот же код ...
    return self
}
```

**Когда использовать сейчас**  
- Поддержка iOS < 11 (редко)  
- Очень специфические формы (например, только один угол)  
- В старых проектах / legacy-коде

### 3. `addBorder(color:width:)`

**Что делает**  
Просто устанавливает `borderColor` и `borderWidth` на layer.

**Важные нюансы**  
- Работает только если **нет маски** (`layer.mask == nil`)  
- Если вы вызвали `roundCorners(…)` с маской → граница исчезнет (маска обрезает border)  
- Border рисуется **внутри** bounds, а не снаружи

**Альтернатива для границ + закруглённых углов + тени** (самый популярный паттерн 2024–2026)

```swift
// Вариант 1 — shadow отдельным слоем
func applyCardStyle(radius: CGFloat = 16, borderWidth: CGFloat = 1, borderColor: UIColor = .gray) {
    roundAllCorners(radius: radius)           // или maskedCorners
    addBorder(color: borderColor, width: borderWidth)
    
    layer.shadowColor = UIColor.black.cgColor
    layer.shadowOpacity = 0.12
    layer.shadowOffset = CGSize(width: 0, height: 2)
    layer.shadowRadius = 8
    
    // для производительности
    layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: radius).cgPath
}
```

### Сравнительная таблица (2025–2026 реалии)

| Метод                  | Выборочные углы | Тень одновременно | Производительность | Offscreen rendering | Рекомендация сейчас          |
|------------------------|------------------|---------------------|---------------------|-----------------------|-------------------------------|
| `roundAllCorners`      | Нет (все или с maskedCorners) | Да (если !masksToBounds) | Отличная           | Обычно нет           | Основной выбор                |
| `roundCorners` (mask)  | Да               | Нет (обрезает тень) | Средняя / плохая   | Да                   | Только если < iOS 11 или очень нужно |
| `addBorder`            | —                | —                   | Отличная           | Нет                  | Использовать с `cornerRadius` |

### Пример использования (современный стиль)

```swift
class ModernCardView: UIView {
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        layer.cornerRadius = 20
        layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner] // только верх
        
        layer.borderWidth = 1
        layer.borderColor = UIColor.systemGray4.cgColor
        
        layer.shadowColor = UIColor.black.cgColor
        layer.shadowOpacity = 0.1
        layer.shadowRadius = 10
        layer.shadowOffset = .zero
        layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: 20).cgPath
    }
}
```

