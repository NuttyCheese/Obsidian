**UIEvent** — это базовый класс в UIKit, представляющий **любое событие**, связанное с взаимодействием пользователя с устройством (касания, движения, нажатия кнопок, shake, внешние события и т.д.).

Все конкретные события (касания, движения, нажатия клавиш и т.д.) наследуются от `UIEvent`.

### Основные типы событий (UIEvent.EventType)

| Тип события (EventType)          | Когда возникает                                      | Самый частый сценарий в 2026 году                     | Класс события |
|----------------------------------|------------------------------------------------------|-------------------------------------------------------|---------------|
| `.touches`                       | Касания пальцами (touch began/moved/ended/cancelled) | Почти все жесты, drag, tap, long press                | `UITouch`     |
| `.motion`                        | Движение устройства (shake, ускорение)               | Shake to undo, встряхивание для случайного выбора     | `UIEvent`     |
| `.remoteControl`                 | Управление с наушников / пульта / кнопок наушников   | Play/pause, next track, volume с гарнитуры            | `UIEvent`     |
| `.keyboard` (iOS 13.4+)          | Нажатия клавиш внешней клавиатуры                    | Кастомные шорткаты в приложении                       | `UIEvent`     |
| `.gameController` (редко)        | События от геймпада / контроллера                    | Игры, симуляторы                                      | `UIEvent`     |

### Самые важные свойства и методы UIEvent

| Свойство / Метод                  | Тип / Возвращает                                     | Что возвращает / зачем нужно                                  | Самый частый сценарий |
|-----------------------------------|-------------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `type`                            | `UIEvent.EventType`                                   | Тип события (touches, motion, remoteControl и т.д.)           | Проверка, с каким событием работаем |
| `subtype`                         | `UIEvent.EventSubtype`                                | Дополнительная информация (например, shake, music control)    | `.motionShake` для shake to undo |
| `timestamp`                       | `TimeInterval`                                        | Время возникновения события (в секундах с 1970)               | Логи, отладка, фильтрация |
| `allTouches`                      | `Set<UITouch>?`                                       | Все текущие касания на экране                                 | Мультитач, жесты двумя руками |
| `touches(for:)`                   | `Set<UITouch>?`                                       | Касания, относящиеся к конкретному view                       | `touches(for: view)` в `touchesBegan` |
| `coalescedTouches(for:)`          | `Set<UITouch>?` (iOS 9+)                              | Коалесцированные (объединённые) касания для высокой частоты   | Рисование, быстрые движения |
| `predictedTouches(for:)`          | `Set<UITouch>?` (iOS 9+)                              | Предсказанные будущие касания (Apple Pencil, finger)          | Плавное рисование, предсказание траектории |

### Самые частые сценарии использования UIEvent в 2026 году

#### 1. Отслеживание касаний в кастомном UIView (переопределение методов)

```swift
class CustomDrawingView: UIView {
    
    private var path = UIBezierPath()
    private var currentTouch: UITouch?
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        currentTouch = touch
        
        let location = touch.location(in: self)
        path.move(to: location)
        setNeedsDisplay()
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first,
              touch === currentTouch else { return }
        
        let location = touch.location(in: self)
        path.addLine(to: location)
        setNeedsDisplay()
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        currentTouch = nil
    }
    
    override func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent?) {
        currentTouch = nil
        path.removeAllPoints()
        setNeedsDisplay()
    }
    
    override func draw(_ rect: CGRect) {
        UIColor.systemBlue.setStroke()
        path.lineWidth = 4
        path.stroke()
    }
}
```

#### 2. Shake to undo / shake to refresh (самый классический motion event)

```swift
class ShakeViewController: UIViewController {
    
    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            // Пользователь встряхнул устройство
            showUndoAlert()
        }
    }
    
    private func showUndoAlert() {
        let alert = UIAlertController(title: "Отменить действие?", message: nil, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "Отменить", style: .default))
        alert.addAction(UIAlertAction(title: "Нет", style: .cancel))
        present(alert, animated: true)
    }
}
```

#### 3. Отслеживание удалённого управления (наушники, пульт)

```swift
class MusicPlayerViewController: UIViewController {
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        becomeFirstResponder()  // Важно!
    }
    
    override var canBecomeFirstResponder: Bool {
        true
    }
    
    override func remoteControlReceived(with event: UIEvent?) {
        guard let event = event, event.type == .remoteControl else { return }
        
        switch event.subtype {
        case .remoteControlPlay:
            playMusic()
        case .remoteControlPause:
            pauseMusic()
        case .remoteControlNextTrack:
            nextTrack()
        case .remoteControlPreviousTrack:
            previousTrack()
        default:
            break
        }
    }
    
    private func playMusic() { /* ... */ }
    private func pauseMusic() { /* ... */ }
    // ...
}
```

### Лучшие практики работы с UIEvent в 2026 году

- **Всегда** переопределяйте методы `touchesBegan/Moved/Ended/Cancelled` в подклассе `UIView`, если нужно кастомное поведение касаний  
- **Для shake** — переопределяйте `motionEnded` и не забывайте `becomeFirstResponder()`  
- **Для remote control** — реализуйте `remoteControlReceived` + `canBecomeFirstResponder` + `becomeFirstResponder`  
- **Для мультитач** — используйте `allTouches` и `touches(for:)`  
- **Для Apple Pencil** — проверяйте `type == .pencil` и `estimatedPropertiesExpectingUpdates`  
- **Для SwiftUI** — используйте `onTapGesture`, `DragGesture`, `LongPressGesture` — UIEvent нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    // Начало рисования или жеста
}
```

**Короткий итог 2026**:
> `UIEvent` — базовый класс для **всех событий пользователя** в UIKit (касания, shake, remote control и т.д.).  
> В 2026 году:  
> - самый частый тип — `.touches` (через `UITouch`)  
> - ключевые методы — `touchesBegan/Moved/Ended/Cancelled`, `motionEnded`, `remoteControlReceived`  
> - для жестов в SwiftUI используйте `Gesture` — UIEvent нужен только в UIKit  
> - это **фундаментальный** класс для любого кастомного взаимодействия с экраном  

Удачи с плавными и отзывчивыми жестами в твоём приложении! 👆