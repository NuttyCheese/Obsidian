**TouchUpOutside** — это событие `UIControl.Event.touchUpOutside`, которое срабатывает **только в одном конкретном случае**:

Пользователь **нажал** на элемент управления (чаще всего кнопку), **удерживал палец**, но **отпустил его за пределами границ** этого элемента.

Это **отмена нажатия** — пользователь передумал, свайпнул мимо или просто промахнулся при отпускании.

### Почему TouchUpOutside так важен в UX 2026 года

| Ситуация / поведение пользователя             | Какое событие приходит                  | Что нужно сделать в приложении (рекомендация 2026) |
|------------------------------------------------|------------------------------------------|-----------------------------------------------------|
| Нажал → отпустил внутри кнопки                 | `touchUpInside`                          | Выполнить действие (логин, купить, сохранить)       |
| Нажал → отпустил **вне** кнопки                | `touchUpOutside`                         | **Отменить** эффект нажатия, вернуть кнопку в исходное состояние |
| Нажал → начал свайп за пределы → отпустил      | `touchDragExit` → `touchUpOutside`       | Отменить выделение, сбросить анимацию               |
| Нажал → система прервала (звонок, жест назад) | `touchCancel`                            | Откатить состояние, убрать выделение                |
| Нажал → держит палец долго                     | `touchDown` → ничего до отпускания       | Показать эффект удержания (если нужно)             |

**Самая частая и правильная последовательность при "плохом" нажатии**:
```
touchDown → touchDragInside → touchDragExit → touchUpOutside
```

### Когда и зачем используют именно TouchUpOutside в 2026 году

| Цель / Эффект                                 | Почему именно `touchUpOutside`                          | Пример кода (коротко) |
|-----------------------------------------------|--------------------------------------------------------|-----------------------|
| **Отмена визуального эффекта нажатия**        | Пользователь передумал — нужно вернуть кнопку в нормальный вид | `sender.backgroundColor = .systemBlue` |
| Сброс анимации scale / shadow / alpha         | Кнопка не должна оставаться "нажатой"                  | `sender.transform = .identity` |
| Отмена предварительной логики                 | Например, начал предзагрузку на `touchDown` — отменить | `cancelPreload()` |
| Избежать случайных действий                   | Если палец сместился — не выполнять основное действие  | Не вызывать `login()` |
| Улучшить ощущение "живой" кнопки              | Пользователь видит, что система "поняла" отмену        | Плавный возврат цвета/размера |
| Комбинировать с long press / context menu     | Долгое нажатие → меню, короткое + уход за пределы → ничего | `UIMenu` на `longPress`, отмена на `touchUpOutside` |

### Самый современный и рекомендуемый паттерн 2026 года (полный контроль UX кнопки)

```swift
final class ModernButton: UIButton {
    
    private var initialBackgroundColor: UIColor!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        layer.cornerRadius = 16
        titleLabel?.font = .systemFont(ofSize: 17, weight: .semibold)
        initialBackgroundColor = backgroundColor ?? .systemBlue
        
        // Добавляем все нужные обработчики
        addTarget(self, action: #selector(touchDown), for: .touchDown)
        addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
        addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
        addTarget(self, action: #selector(touchDragEnter), for: .touchDragEnter) // для возврата эффекта
    }
    
    @objc private func touchDown() {
        UIView.animate(withDuration: 0.12) {
            self.transform = CGAffineTransform(scaleX: 0.94, y: 0.94)
            self.backgroundColor = self.initialBackgroundColor.withAlphaComponent(0.85)
            self.layer.shadowOpacity = 0.4
        }
        
        // Лёгкий haptic
        UIImpactFeedbackGenerator(style: .medium).impactOccurred()
    }
    
    @objc private func touchUp() {
        UIView.animate(withDuration: 0.22, delay: 0, usingSpringWithDamping: 0.75, initialSpringVelocity: 0.4) {
            self.transform = .identity
            self.backgroundColor = self.initialBackgroundColor
            self.layer.shadowOpacity = 0
        }
    }
    
    @objc private func touchDragExit() {
        // Палец ушёл за пределы — отменяем эффект
        UIView.animate(withDuration: 0.15) {
            self.transform = .identity
            self.backgroundColor = self.initialBackgroundColor
            self.layer.shadowOpacity = 0
        }
    }
    
    @objc private func touchDragEnter() {
        // Палец вернулся в границы — возвращаем эффект нажатия
        touchDown()
    }
}
```

### Почему именно такой подход в 2026 году

- **0.12 сек** на нажатие + **0.22 сек spring** на возврат — идеальное время по Human Interface Guidelines  
- **Haptic feedback** (medium) — делает кнопку "живой", как системные элементы iOS  
- **touchDragExit + touchDragEnter** — полный контроль за поведением при свайпе пальцем  
- **touchCancel** — обрабатывает прерывания (входящий звонок, жест назад, системные события)  
- **Нет reliance на `highlighted`** — Apple рекомендует кастомные анимации вместо стандартного состояния

### Полный список событий для идеальной кнопки 2026

```swift
button.addTarget(self, action: #selector(touchDown), for: .touchDown)
button.addTarget(self, action: #selector(touchDragEnter), for: .touchDragEnter)
button.addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
button.addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
```

Но в 90% случаев достаточно:

- `.touchDown` → начать эффект  
- `.touchUpInside` → выполнить действие  
- `.touchUpOutside` + `.touchCancel` + `.touchDragExit` → отменить эффект

### Лучшие практики TouchUpInside в Swift 2026

- **Делай анимацию короткой** (0.1–0.25 сек) и естественной (spring damping 0.7–0.9)  
- **Используй UIImpactFeedbackGenerator** (medium/light) для тактильного отклика  
- **Не перегружай обработчик** — избегай сети, тяжёлых вычислений в `touchUpInside`  
- **Комбинируй с `highlighted`** — если нужен быстрый fallback  
- **[[@MainActor]]** — все обработчики событий UI — на главном акторе  
- **[[Swift]] 6 strict concurrency** — события вызываются на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «@objc func — основное действие кнопки при успешном [[TouchUpInside]]»

**Короткий девиз 2026**:
> `TouchUpInside` — это **самое надёжное** событие для выполнения действия кнопки: пользователь **нажал и отпустил внутри** → значит, он **точно хотел** это сделать.  
> В 2026 году это **основной** триггер для всех важных действий (логин, отправить, купить, перейти).  
> Всегда комбинируй с `touchDown` (эффект нажатия) и `touchUpOutside`/`touchCancel` (отмена эффекта).
