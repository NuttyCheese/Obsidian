**`didMoveToSuperview`** и **`didMoveToWindow`** — это два метода жизненного цикла [[UIView]] в [[UIKit]], которые вызываются, когда представление **добавляется** или **удаляется** из иерархии представлений.

Они очень полезны для выполнения действий, зависящих от того, находится ли вью на экране или нет.

### Основные методы

| Метод                  | Когда вызывается                                                                | Самые частые сценарии использования в 2026                                        | Важные замечания                          |
| ---------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ----------------------------------------- |
| `didMoveToSuperview()` | После того, как вью добавили или удалили из superview (т.е. изменился родитель) | Настройка наблюдателей, начало/остановка анимаций, [[KVO]], добавление subviews   | Вызывается **до** `didMoveToWindow`       |
| `didMoveToWindow()`    | После того, как вью добавили в окно (window) или удалили из него                | Запуск/остановка таймеров, сетевых запросов, анимаций, [[AVPlayer]], [[SceneKit]] | Вызывается **после** `didMoveToSuperview` |

### Порядок вызовов при добавлении/удалении

#### Добавление в иерархию

1. `willMoveToSuperview(newSuperview)`
2. добавление в superview
3. `didMoveToSuperview()` — superview уже новый
4. `willMoveToWindow(newWindow)`
5. добавление в окно
6. `didMoveToWindow()` — window уже новый

#### Удаление из иерархии

1. `willMoveToSuperview(nil)`
2. удаление из superview
3. `didMoveToSuperview()` — superview = nil
4. `willMoveToWindow(nil)`
5. удаление из окна
6. `didMoveToWindow()` — window = nil

### Самые популярные и рекомендуемые паттерны в 2026 году

#### 1. Запуск/остановка анимации или таймера (самый частый случай)

```swift
class AnimatedView: UIView {
    
    private var timer: Timer?
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        if window != nil {
            // Вью появилось на экране → запускаем анимацию / таймер
            startAnimation()
        } else {
            // Вью ушло с экрана → останавливаем
            stopAnimation()
        }
    }
    
    private func startAnimation() {
        timer = Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { _ in
            // анимация
        }
    }
    
    private func stopAnimation() {
        timer?.invalidate()
        timer = nil
    }
}
```

#### 2. Регистрация/отмена [[KVO]] или [[NotificationCenter]]

```swift
class ProfileHeaderView: UIView {
    
    private var observation: NSKeyValueObservation?
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        if window != nil {
            // Подписываемся только когда вью на экране
            observation = User.current.observe(\.avatarURL) { [weak self] _, _ in
                self?.updateAvatar()
            }
        } else {
            // Отписываемся, чтобы не было утечек
            observation?.invalidate()
            observation = nil
        }
    }
}
```

#### 3. Проверка, когда вью впервые появилось на экране

```swift
class CustomCell: UITableViewCell {
    
    private var hasAppeared = false
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        if window != nil && !hasAppeared {
            hasAppeared = true
            // Выполняем действие только один раз при первом появлении
            startIntroAnimation()
        }
    }
}
```

### Лучшие практики didMoveToSuperview / didMoveToWindow в Swift 2026

- **Предпочитайте** `didMoveToWindow()` — он точнее определяет, виден ли вью пользователю (на экране или нет)  
- **Используйте** `didMoveToSuperview()` — когда логика зависит именно от наличия superview (например, добавление/удаление subviews)  
- **Всегда** вызывайте `super.didMoveTo...()` — особенно в подклассах UIView / UIViewController  
- **Не делайте** тяжёлые операции (сеть, диск, сложные вычисления) — метод может вызываться очень часто  
- **Для остановки ресурсов** — проверяйте `window == nil` или `superview == nil`  
- **В SwiftUI** — эти методы **не используются** (аналог — `.onAppear` / `.onDisappear` или `UIViewRepresentable`)  
- **Документируйте** — пишите комментарий «didMoveToWindow — запуск/остановка таймера и анимации при появлении/исчезновении вью»

**Короткий итог 2026**:
> `didMoveToSuperview()` и `didMoveToWindow()` — это **жизненный цикл**, который сообщает, когда вью **добавили/удалили** из иерархии или окна.  
> В 2026 году:  
> - используйте `didMoveToWindow()` для запуска/остановки ресурсов (таймеры, анимации, подписки)  
> - используйте `didMoveToSuperview()` для работы с superview  
> - проверяйте `window != nil` / `superview != nil`  
> - вызывайте `super`  
> Это **очень надёжный** способ избежать утечек памяти и ненужных фоновых операций.
