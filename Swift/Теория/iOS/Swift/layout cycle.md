**Layout cycle** (цикл компоновки) в UIKit — это внутренний механизм, который отвечает за **определение размеров и позиций** всех представлений (views) на экране.  
Он запускается автоматически, когда система понимает, что текущая раскладка может быть уже неактуальной.

### Когда запускается layout cycle (основные триггеры)

1. **Изменение размеров экрана / ориентации**  
   → `viewWillTransition(to:with:)`, `traitCollectionDidChange`, Safe Area insets

2. **Изменение размера или позиции superview**  
   → вызов `setNeedsLayout()` / `layoutIfNeeded()` выше по иерархии

3. **Добавление / удаление subview**  
   → `addSubview`, `removeFromSuperview`

4. **Изменение intrinsic content size**  
   → изменение текста в UILabel, загрузка изображения в UIImageView

5. **Изменение constraints**  
   → активация/деактивация, изменение constant, priority

6. **Вызов методов, которые явно требуют перекомпоновки**  
   - `setNeedsLayout()` — помечает, что нужно перекомпоновать (отложенно)  
   - `layoutIfNeeded()` — выполняет layout **немедленно** (синхронно)

7. **Изменение safe area insets**, content inset, scroll view zoom и т.д.

### Основные методы жизненного цикла layout

| Метод                        | Когда вызывается                              | Что обычно делают внутри                     | Можно ли безопасно менять constraints? |
|------------------------------|-----------------------------------------------|----------------------------------------------|----------------------------------------|
| `layoutSubviews()`           | Самый важный — здесь происходит реальная компоновка | Ручная установка frame, позиционирование subviews | **Очень осторожно** (часто приводит к cycle) |
| `updateConstraints()`        | Перед layoutSubviews, если нужно обновить constraints | Добавление/изменение constraints             | **Да, это самое безопасное место**     |
| `setNeedsUpdateConstraints()`| Помечает, что constraints нужно пересчитать    | —                                            | —                                      |
| `setNeedsLayout()`           | Помечает, что нужно вызвать layoutSubviews     | —                                            | —                                      |
| `layoutIfNeeded()`           | Выполняет layout немедленно (синхронно)        | —                                            | —                                      |

### Самые частые причины layout cycle (бесконечный цикл компоновки)

1. **В `layoutSubviews()` меняешь constraints или вызываешь `setNeedsLayout()` / `layoutIfNeeded()`**  
   → классическая ошибка

2. **В `updateConstraints()` вызываешь `layoutIfNeeded()` или меняешь frame напрямую**

3. **Авто-ресайз subview в зависимости от superview, а superview — от subview**  
   (взаимная зависимость)

4. **Использование `intrinsicContentSize` + ручное изменение frame в `layoutSubviews()`**

5. **Анимация + изменение constraints внутри анимационного блока без правильной последовательности**

### Как обнаружить и исправить layout cycle

#### Способы обнаружения (2025–2026)

1. Включить **Debug → View Debugging → Show Layout Issues** в симуляторе/Xcode  
   → Xcode подсветит проблемные constraints красным

2. Логировать вызовы

```swift
override func layoutSubviews() {
    super.layoutSubviews()
    print("layoutSubviews called on \(type(of: self)) – frame = \(frame)")
}
```

3. Использовать **Instrument → Time Profiler** + фильтр по `layoutSubviews` / `updateConstraints`

4. Включить **SwiftUI Inspector** (если смешанный UIKit + SwiftUI) — он часто показывает конфликты

#### Типичные исправления

- **Никогда** не вызывай `setNeedsLayout()`, `layoutIfNeeded()`, `setNeedsUpdateConstraints()` внутри `layoutSubviews()`

- **Не меняй constraints** внутри `layoutSubviews()` — делай это в `updateConstraints()`

- **Используй `invalidateIntrinsicContentSize()`** вместо ручного изменения frame

- **Для анимации constraints** — используй `UIView.animate` + `layoutIfNeeded()` **внутри** анимационного блока

```swift
UIView.animate(withDuration: 0.3) {
    self.someConstraint.constant = 100
    self.view.layoutIfNeeded()
}
```

- **Для динамического контента** — переопредели `intrinsicContentSize` и вызови `invalidateIntrinsicContentSize()`

### Самый надёжный паттерн 2026 года (UIKit)

```swift
class CustomView: UIView {
    
    private lazy var label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        addSubview(label)
        // начальные constraints
    }
    
    override func updateConstraints() {
        super.updateConstraints()
        
        // все constraints здесь
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.topAnchor.constraint(equalTo: topAnchor, constant: 16),
            label.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            label.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16)
        ])
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        // только если нужно что-то посчитать на основе реальных размеров
        // НЕ меняй здесь constraints!
    }
}
```

### Короткий чек-лист «нет ли layout cycle?»

- Вызываешь ли `setNeedsLayout()` / `layoutIfNeeded()` внутри `layoutSubviews()`? → **нет**
- Меняешь ли constraints внутри `layoutSubviews()`? → **нет**
- Есть ли взаимозависимость между superview и subview? → **проверь**
- Используешь ли `invalidateIntrinsicContentSize()` вместо ручного frame? → **да**
- Анимации constraints делаешь через `layoutIfNeeded()` внутри `animate`? → **да**

**Короткий девиз 2026**:
> Layout cycle — это когда view постоянно переспрашивает «а как мне теперь расположиться?» и не может остановиться.  
> В 2026 году:  
> - constraints → только в `updateConstraints()`  
> - ручная позиция → только в `layoutSubviews()` (и очень редко)  
> - анимация → `layoutIfNeeded()` внутри `animate`  
> - всё остальное → Auto Layout + `invalidateIntrinsicContentSize()`

Удачи с ровной, предсказуемой и быстрой компоновкой в твоём приложении! 📐