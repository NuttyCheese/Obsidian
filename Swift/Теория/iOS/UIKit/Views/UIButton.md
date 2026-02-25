**UIButton** — это основной класс в [[UIKit]] для создания **интерактивных кнопок** в [[iOS]]-приложениях. Это подкласс `UIControl`, поэтому он поддерживает **target-action** механизм, различные состояния (normal, highlighted, selected, disabled) и множество событий (touch, drag, value changed и т.д.).

В 2026 году это **самый распространённый** элемент управления для любых действий пользователя: отправить форму, перейти на экран, запустить процесс, лайкнуть, добавить в избранное и т.д.

### 1. Почему UIButton — это основа интерфейса iOS

| Характеристика                            | Почему это важно в 2026 году                                                                  | Пример использования                                            |
| ----------------------------------------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| **Нативный внешний вид**                  | Автоматически адаптируется под iOS 18/19 дизайн (Dynamic Island, tinted icons, SF Symbols 6+) | Системные кнопки выглядят как в Settings, Photos, Messages      |
| **Поддержка состояний**                   | `.normal`, `.highlighted`, `.selected`, `.disabled` — всё нативно                             | Кнопка "сжимается" при нажатии, становится серой при отключении |
| **Гибкие события**                        | `.touchUpInside`, `.touchDown`, `.touchUpOutside`, `.touchCancel` и др.                       | Полный контроль над UX нажатия                                  |
| **SF Symbols + tintColor**                | Полная поддержка цветовых тем, weight, scale, multicolor                                      | Кнопки с иконками адаптируются под light/dark/accent            |
| **[[UIAction]] (iOS 14+)**                | Замыкания вместо `#selector` — чистый код без `@objc`                                         | Современный стандарт 2026 года                                  |
| **Доступность (VoiceOver, Dynamic Type)** | Всё работает из коробки (accessibilityLabel, traits)                                          | Обязательно для App Store                                       |

### 2. Основные способы создания UIButton в 2026 году

| Способ создания                               | Когда использовать                                   | Пример кода (коротко) |
|-----------------------------------------------|------------------------------------------------------|-----------------------|
| **Программно — .system**                      | Стандартные кнопки с синим текстом                   | `UIButton(type: .system)` |
| **Программно — .custom**                      | Полный контроль (иконка без текста, фон, shadow)     | `UIButton(type: .custom)` |
| **Storyboard / XIB**                          | Быстрый прототип, legacy-проекты                     | `@IBOutlet weak var button: UIButton!` |
| **UIAction + primaryAction** (iOS 14+)        | Современный стиль без `#selector`                    | `button.addAction(UIAction { ... }, for: .touchUpInside)` |
| **Кастомный подкласс UIButton**               | Сложная логика, анимации, кастомные состояния       | `class PrimaryButton: UIButton { ... }` |

### 3. Полный современный паттерн 2026 года (рекомендуемый)

```swift
final class PrimaryButton: UIButton {
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        // Внешний вид
        backgroundColor = .systemBlue
        setTitleColor(.white, for: .normal)
        titleLabel?.font = .systemFont(ofSize: 17, weight: .semibold)
        layer.cornerRadius = 12
        layer.shadowColor = UIColor.black.cgColor
        layer.shadowOpacity = 0.25
        layer.shadowOffset = CGSize(width: 0, height: 4)
        layer.shadowRadius = 8
        
        // Современный способ привязки действия
        addAction(UIAction { [weak self] _ in
            self?.buttonTapped()
        }, for: .touchUpInside)
        
        // Эффект нажатия (touchDown / touchUp)
        addAction(UIAction { [weak self] _ in
            UIView.animate(withDuration: 0.12) {
                self?.transform = CGAffineTransform(scaleX: 0.94, y: 0.94)
            }
        }, for: .touchDown)
        
        addAction(UIAction { [weak self] _ in
            UIView.animate(withDuration: 0.22, delay: 0, usingSpringWithDamping: 0.75, initialSpringVelocity: 0.4) {
                self?.transform = .identity
            }
        }, for: .touchUpInside)
    }
    
    private func buttonTapped() {
        print("Кнопка нажата!")
        // Основное действие
    }
}
```

### 4. Состояния кнопки и как их настраивать

| Состояние         | Когда активно                                   | Как настроить в 2026 |
|-------------------|-------------------------------------------------|----------------------|
| `.normal`         | Обычное состояние                               | `setTitle("Tap", for: .normal)` |
| `.highlighted`    | Пока палец нажат (touchDown)                    | `setTitleColor(.gray, for: .highlighted)` |
| `.selected`       | Кнопка выбрана (toggle)                         | `isSelected = true` + `setImage(..., for: .selected)` |
| `.disabled`       | `isEnabled = false`                             | `setTitleColor(.gray, for: .disabled)` |

### 5. Лучшие практики UIButton в Swift 2026

- **Используй UIAction вместо target-action** — это стандарт с iOS 14+  
- **Не используй .custom без необходимости** — `.system` уже адаптируется под тему  
- **SF Symbols** — всегда с `weight` и `scale` для лучшей читаемости  
- **Анимация нажатия** — scale 0.94–0.96 + лёгкий opacity + haptic (UIImpactFeedbackGenerator)  
- **Доступность** — обязательно задавай `accessibilityLabel`, `accessibilityHint`  
- **@MainActor** — все действия кнопок — на главном акторе  
- **Swift 6 strict concurrency** — UIButton полностью безопасен  
- **Документируйте** — пиши комментарий «PrimaryButton — кастомная кнопка с UIAction и анимацией нажатия»

**Короткий девиз 2026**:
> UIButton — это **универсальная кнопка** UIKit: текст, иконка, фон, состояния, анимации, действия.  
> В 2026 году используй **UIAction**, **SF Symbols**, **кастомный подкласс** и **анимацию нажатия**.  
> Это **единственный правильный** способ создать кнопку в UIKit.
