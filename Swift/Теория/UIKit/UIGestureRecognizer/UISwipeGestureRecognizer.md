**UISwipeGestureRecognizer** — это жест из [[UIKit]], который распознаёт **быстрый свайп** (резкое проведение пальцем) в одном из четырёх основных направлений: влево, вправо, вверх или вниз.

Он идеально подходит для реализации **быстрых действий**, связанных с направлением движения пальца, без необходимости отслеживать полное перетаскивание (как в UIPanGestureRecognizer).

### Для чего используют UISwipeGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                      | Направление свайпа | Пример в приложениях 2026 |
|-----------------------------------------------|---------------------|----------------------------|
| Переход назад / вперёд (навигация)            | ← (left) / → (right) | Свайп влево → назад (альтернатива системному жесту) |
| Swipe-to-delete / архивировать / завершить    | ← (left) / → (right) | Свайп влево на задаче → удалить / завершить |
| Свайп для переключения вкладок / страниц      | ← / →               | Кастомные таб-бары, onboarding, галерея фото |
| Свайп вверх/вниз для дополнительных действий  | ↑ (up) / ↓ (down)   | Свайп вверх → показать скрытый контент / меню |
| Быстрый лайк / дизлайк / шаринг               | → (right) / ← (left) | Tinder-подобные приложения, быстрые реакции в чатах |
| Свайп для открытия бокового меню              | → (right)           | Hamburger-меню (хотя чаще используют UIScreenEdgePan) |
| Свайп для pull-to-refresh (редко)             | ↓ (down)            | Кастомный рефреш сверху |

### Что необходимо сделать, чтобы начать использовать UISwipeGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + @MainActor)

```swift
import UIKit

final class ChatViewController: UIViewController {
    
    private let messageView: UIView = {
        let v = UIView()
        v.backgroundColor = .systemBlue
        v.layer.cornerRadius = 12
        v.translatesAutoresizingMaskIntoConstraints = false
        return v
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .systemBackground
        view.addSubview(messageView)
        
        NSLayoutConstraint.activate([
            messageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            messageView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            messageView.widthAnchor.constraint(equalToConstant: 200),
            messageView.heightAnchor.constraint(equalToConstant: 100)
        ])
        
        setupSwipeGestures()
    }
    
    private func setupSwipeGestures() {
        // 1. Свайп вправо (например, архивировать)
        let swipeRight = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
        swipeRight.direction = .right
        swipeRight.numberOfTouchesRequired = 1
        
        // 2. Свайп влево (например, удалить)
        let swipeLeft = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
        swipeLeft.direction = .left
        swipeLeft.numberOfTouchesRequired = 1
        
        // 3. Добавляем оба жеста к нужной вью
        messageView.addGestureRecognizer(swipeRight)
        messageView.addGestureRecognizer(swipeLeft)
        
        messageView.isUserInteractionEnabled = true
    }
    
    @objc private func handleSwipe(_ gesture: UISwipeGestureRecognizer) {
        let direction: String
        
        switch gesture.direction {
        case .right:
            direction = "вправо"
            // Например: архивировать сообщение
            UIView.animate(withDuration: 0.3) {
                self.messageView.transform = CGAffineTransform(translationX: 300, y: 0)
            } completion: { _ in
                // Удалить или скрыть вью
            }
            
        case .left:
            direction = "влево"
            // Например: удалить сообщение
            UIView.animate(withDuration: 0.3) {
                self.messageView.transform = CGAffineTransform(translationX: -300, y: 0)
            }
            
        case .up:
            direction = "вверх"
        case .down:
            direction = "вниз"
        default:
            direction = "неизвестно"
        }
        
        print("Свайп обнаружен: \(direction)")
    }
}
```

### Лучшие практики UISwipeGestureRecognizer в Swift 2026

- **direction** — обязательно указывай (.left, .right, .up, .down или комбинацию через [.left, .right])  
- **numberOfTouchesRequired = 1** — почти всегда (для мультитач используй 2+)  
- **gesture.state == .ended** — основной момент для принятия решения (в .began/.changed можно отслеживать preview)  
- **velocity(in:)** — если нужно проверить скорость свайпа (например, > 500 для "быстрого")  
- **@objc** — обязательно для метода-обработчика  
- **@MainActor** — все обработчики жестов — на главном акторе  
- **Swift 6 strict concurrency** — UIGestureRecognizer полностью безопасен  
- **Документируйте** — пиши комментарий «UISwipeGestureRecognizer — свайп влево для удаления сообщения»

**Короткий девиз 2026**:
> UISwipeGestureRecognizer — это когда тебе нужно быстро реагировать на **резкий свайп в конкретном направлении** (влево, вправо, вверх, вниз).  
> В 2026 году это **основной жест** для swipe-to-delete, навигации, меню и быстрых действий в списках и карточках.  
> Всегда указывай direction и проверяй .ended для финального действия.

Удачи с быстрыми и интуитивными свайпами в Swift! 👉