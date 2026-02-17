**TouchUpInside** — это одно из самых популярных и надёжных событий (`UIControl.Event.touchUpInside`) в UIKit, которое срабатывает **только тогда, когда пользователь нажал на элемент управления (чаще всего кнопку) и отпустил палец внутри его границ**.

Это **основное событие** для обработки нажатия кнопок в iOS-приложениях.

### Почему именно TouchUpInside, а не другие события

| Событие                | Когда срабатывает                                      | Надёжность для действий | Самый частый сценарий 2026 |
|------------------------|--------------------------------------------------------|--------------------------|----------------------------|
| `touchDown`            | Палец коснулся кнопки                                  | Низкая (может быть отменено) | Визуальный эффект нажатия (scale, цвет) |
| `touchUpInside`        | Палец отпущен **внутри** кнопки                        | **Высочайшая**           | Основное действие (логин, отправить, сохранить) |
| `touchUpOutside`       | Палец отпущен **вне** кнопки                           | Средняя                  | Отмена эффекта нажатия |
| `touchCancel`          | Система отменила касание (звонок, жест назад)          | Средняя                  | Откат состояния |
| `touchDragExit`        | Палец вышел за границы кнопки                          | Низкая                   | Отмена выделения |

**Самая частая и безопасная последовательность** при обычном нажатии кнопки:
```
touchDown → (touchDragInside, если двигал пальцем) → touchUpInside
```

Если пользователь свайпнул за пределы кнопки → вместо `touchUpInside` придёт `touchUpOutside` или `touchDragExit`.

### Для чего используют TouchUpInside в 2026 году (реальные сценарии)

| Цель / Действие                               | Почему именно `touchUpInside`                          | Пример кода (коротко) |
|-----------------------------------------------|--------------------------------------------------------|-----------------------|
| Основное действие кнопки (логин, отправить, купить) | Гарантия, что пользователь **осознанно** нажал и отпустил внутри | `loginButton.addTarget(self, action: #selector(login), for: .touchUpInside)` |
| Запуск анимации / вибрации / звука после подтверждённого нажатия | Эффект должен быть только при успешном нажатии          | `sender.playHapticFeedback()` |
| Сохранение формы / отправка данных            | Действие дорогое → делаем только при уверенном тапе    | `saveFormData()` |
| Переход на следующий экран                    | `pushViewController` или `present`                     | `navigationController?.pushViewController(detailVC, animated: true)` |
| Toggle состояния (лайк, избранное)            | Изменение состояния только при полном нажатии          | `isLiked.toggle()` |
| Запуск длительной операции (загрузка, запрос) | Пользователь не отменит в процессе                     | `startDownload()` |

### Самый современный и рекомендуемый паттерн 2026 года (с анимацией и haptic)

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
        backgroundColor = .systemBlue
        setTitleColor(.white, for: .normal)
        titleLabel?.font = .systemFont(ofSize: 17, weight: .semibold)
        layer.cornerRadius = 12
        
        // Добавляем обработчики
        addTarget(self, action: #selector(touchDown), for: .touchDown)
        addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
        addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
    }
    
    @objc private func touchDown() {
        UIView.animate(withDuration: 0.12) {
            self.transform = CGAffineTransform(scaleX: 0.94, y: 0.94)
            self.backgroundColor = .systemBlue.withAlphaComponent(0.9)
        }
        
        // Лёгкая вибрация (haptic feedback)
        let generator = UIImpactFeedbackGenerator(style: .medium)
        generator.prepare()
        generator.impactOccurred()
    }
    
    @objc private func touchUp() {
        UIView.animate(withDuration: 0.2, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.5) {
            self.transform = .identity
            self.backgroundColor = .systemBlue
        }
    }
    
    @objc private func touchDragExit() {
        // Если палец вышел за пределы — отменяем эффект
        UIView.animate(withDuration: 0.15) {
            self.transform = .identity
            self.backgroundColor = .systemBlue
        }
    }
}

// Использование в контроллере
class LoginViewController: UIViewController {
    @IBOutlet weak var loginButton: PrimaryButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        loginButton.addTarget(self, action: #selector(loginTapped), for: .touchUpInside)
    }
    
    @objc private func loginTapped() {
        print("Пользователь нажал Войти")
        // Логика авторизации
    }
}
```

### Почему именно такой подход в 2026 году

- **0.12 сек** — оптимальное время для анимации нажатия (по Human Interface Guidelines)  
- **Spring damping 0.7** — даёт приятный «отскок»  
- **Haptic feedback** — делает кнопку «живой» (как системные кнопки iOS)  
- **touchDragExit** — отменяет эффект, если пользователь передумал и свайпнул за пределы  
- **touchCancel** — обрабатывает системные отмены (входящий звонок, жест назад)  

### Полный список событий для кнопки (рекомендуемый набор 2026)

```swift
button.addTarget(self, action: #selector(touchDown), for: .touchDown)
button.addTarget(self, action: #selector(touchUp), for: [.touchUpInside, .touchUpOutside, .touchCancel])
button.addTarget(self, action: #selector(touchDragExit), for: .touchDragExit)
button.addTarget(self, action: #selector(touchDragEnter), for: .touchDragEnter)   // редко
```

Но в 95% случаев достаточно:
- `.touchDown` — начать эффект нажатия  
- `.touchUpInside` — основное действие  
- `.touchUpOutside` + `.touchCancel` — вернуть в исходное состояние

### Лучшие практики TouchUpInside в Swift 2026

- **Делай анимацию короткой** (0.1–0.2 сек) и естественной (spring damping 0.7–0.9)  
- **Используй UIImpactFeedbackGenerator** для haptic (medium или light)  
- **Не перегружай** — избегай сложных вычислений в обработчике  
- **Комбинируй с `highlighted`** — если хочешь стандартный эффект + кастомный  
- **@MainActor** — все обработчики событий UI — на главном акторе  
- **Swift 6 strict concurrency** — события вызываются на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «@objc func — основное действие кнопки при TouchUpInside»

**Короткий девиз 2026**:
> `TouchUpInside` — это **самое надёжное** событие для выполнения основного действия кнопки: пользователь нажал и **отпустил внутри** → значит, он точно хотел нажать.  
> В 2026 году это **основной** триггер для логина, отправки формы, перехода на экран.  
> Всегда комбинируй с `touchDown` для визуального отклика и `touchUpOutside`/`touchCancel` для отмены.

Удачи с отзывчивыми и интуитивными кнопками в твоём приложении! 👆