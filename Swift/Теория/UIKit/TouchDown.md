**TouchDown** — это одно из событий (`UIControl.Event.touchDown`) в [[UIKit]], которое срабатывает **в самый момент, когда пользователь впервые касается** элемента управления (чаще всего [[UIButton]], но также [[UISlider]], [[UISegmentedControl]] и другие наследники [[UIControl]]).

Это **самое раннее** событие в цепочке касаний кнопки.

### Полная цепочка событий касания кнопки (2026 актуально)

| Событие                  | Когда срабатывает                                      | Кол-во вызовов при одном нажатии | Самый частый сценарий использования в 2026 |
|--------------------------|--------------------------------------------------------|----------------------------------|--------------------------------------------|
| `touchDown`              | Палец только что коснулся кнопки                       | 1                                | Визуальный эффект нажатия (scale down, цвет) |
| `touchDownRepeat`        | Двойное/тройное касание (очень редко)                  | 1+                               | Почти не используется                      |
| `touchDragEnter`         | Палец вошёл в границы кнопки после начала вне её       | 0–1                              | Редко (для кастомных drag-жестов)          |
| `touchDragInside`        | Палец двигается внутри кнопки                          | много                            | Редко                                      |
| `touchDragOutside`       | Палец вышел за границы кнопки                          | 0–1                              | Отмена эффекта нажатия                     |
| `touchDragExit`          | Палец вышел за границы и больше не возвращается        | 0–1                              | Отмена выделения                           |
| `touchUpInside`          | Палец отпущен **внутри** кнопки                        | 0–1                              | Основное действие (нажатие кнопки)         |
| `touchUpOutside`         | Палец отпущен **вне** кнопки                           | 0–1                              | Отмена действия                            |
| `touchCancel`            | Система отменила касание (звонок, системный жест)      | 0–1                              | Откат состояния                            |

**Самая частая последовательность при обычном нажатии**:
```
touchDown → (touchDragInside, если палец двигался) → touchUpInside
```

### Для чего используют именно `TouchDown` в 2026 году

| Цель / Эффект                                 | Почему именно `TouchDown` (а не `touchUpInside`) | Пример кода (коротко) |
|-----------------------------------------------|---------------------------------------------------|-----------------------|
| Показать **эффект нажатия** (scale, тень, цвет) | Реакция должна быть мгновенной, как в iOS Human Interface Guidelines | `sender.transform = .init(scaleX: 0.95, y: 0.95)` |
| Начать **предварительную анимацию** или звук   | Пользователь должен почувствовать отклик сразу при касании | `playClickSound()` |
| Выделить кнопку или показать ripple-эффект     | Ripple должен начинаться с момента касания        | `startRippleAnimation(at: touchPoint)` |
| Запустить **предзагрузку** данных или анимацию | Пока палец держат — уже начать подготовку        | `preloadNextScreen()` |
| Отменить эффект при свайпе за пределы          | Комбинируется с `touchDragExit` / `touchUpOutside` | `resetButtonAppearance()` |

### Самый современный и рекомендуемый паттерн 2026 года

```swift
final class ActionButton: UIButton {
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        // Визуальные настройки
        layer.cornerRadius = 12
        backgroundColor = .systemBlue
        setTitleColor(.white, for: .normal)
        titleLabel?.font = .systemFont(ofSize: 17, weight: .semibold)
        
        // Добавляем обработчики событий
        addTarget(self, action: #selector(touchDown), for: .touchDown)
        addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
        addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
    }
    
    @objc private func touchDown() {
        UIView.animate(withDuration: 0.12) {
            self.transform = CGAffineTransform(scaleX: 0.94, y: 0.94)
            self.backgroundColor = .systemBlue.withAlphaComponent(0.85)
            self.layer.shadowOpacity = 0.4
        }
    }
    
    @objc private func touchUp() {
        UIView.animate(withDuration: 0.2, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.5) {
            self.transform = .identity
            self.backgroundColor = .systemBlue
            self.layer.shadowOpacity = 0
        }
    }
    
    @objc private func touchDragExit() {
        UIView.animate(withDuration: 0.15) {
            self.transform = .identity
            self.backgroundColor = .systemBlue
        }
    }
}
```

### Почему именно такой подход в 2026 году

- **0.12–0.2 секунды** — идеальное время для анимации нажатия (Apple Human Interface Guidelines)  
- **Spring with damping** — даёт естественный «отскок»  
- **touchDragExit** — отменяет эффект, если пользователь свайпнул за пределы (как в системных кнопках)  
- **touchCancel** — обрабатывает системные отмены (входящий звонок, жест назад)  
- **Нет `highlighted` состояния** — Apple рекомендует кастомные анимации вместо стандартного highlighted

### Полный список событий для кнопки (рекомендуемый набор 2026)

```swift
button.addTarget(self, action: #selector(touchDown), for: .touchDown)
button.addTarget(self, action: #selector(touchDownRepeat), for: .touchDownRepeat) // редко
button.addTarget(self, action: #selector(touchDragEnter), for: .touchDragEnter)   // редко
button.addTarget(self, action: #selector(touchDragInside), for: .touchDragInside) // редко
button.addTarget(self, action: #selector(touchDragOutside), for: .touchDragOutside)
button.addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
button.addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
```

Но в 90% случаев достаточно:

- `.touchDown` — начать эффект нажатия  
- `.touchUpInside` + `.touchUpOutside` + `.touchCancel` — вернуть в исходное состояние

### Лучшие практики TouchDown в Swift 2026

- **Делай анимацию короткой** (0.1–0.2 сек) и естественной (spring damping 0.7–0.9)  
- **Не перегружай** — избегай сложных вычислений в `touchDown`  
- **Комбинируй с `highlighted`** — если хочешь стандартный эффект + кастомный  
- **[[@MainActor]]** — все обработчики событий UI — на главном акторе  
- **[[Swift]] 6 strict concurrency** — события вызываются на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «@objc func — визуальный эффект нажатия кнопки»

**Короткий девиз 2026**:
> `TouchDown` — это когда пользователь **только коснулся** кнопки.  
> В 2026 году это **основной** триггер для мгновенной визуальной обратной связи: сжатие, подсветка, ripple, звук.  
> Всегда комбинируй с `touchUpInside`/`touchUpOutside`/`touchCancel` для возврата в исходное состояние.
