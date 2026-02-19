**UIScreenEdgePanGestureRecognizer** — это специализированный жест из [[UIKit]], который распознаёт **свайп (быстрое проведение пальцем) начиная с края экрана**.

Он специально создан для реализации **системных жестов навигации** (например, «свайп слева направо → назад»), боковых меню (side menu / drawer) и других действий, привязанных к краям устройства.

### Для чего используют UIScreenEdgePanGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                            | Как именно используется                                        | Пример в приложениях 2026                     |
| ----------------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| Кастомный жест «назад» (back swipe) | Свайп слева направо → popViewController или dismiss            | Замена системного жеста в кастомной навигации |
| Боковое меню (side menu / drawer)   | Свайп справа налево → открыть меню                             | Hamburger-меню, как в старых приложениях      |
| Переключение вкладок / страниц      | Свайп с левого/правого края → переключить tab / page           | Кастомные таб-бары, onboarding                |
| Pull-to-refresh с края              | Свайп сверху вниз с верхнего края                              | Редко, но иногда вместо [[UIRefreshControl]]  |
| Жесты для accessibility / VoiceOver | Доступные жесты с края для людей с ограниченными возможностями | Приложения с улучшенной доступностью          |
| Кастомные навигационные переходы    | Свайп с края → интерактивный transition                        | Кастомные push/pop анимации                   |

### Что необходимо сделать, чтобы начать использовать UIScreenEdgePanGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + [[@MainActor]])

```swift
import UIKit

final class CustomNavigationViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .systemBackground
        
        // 1. Создаём жест свайпа с левого края (назад)
        let leftEdgePan = UIScreenEdgePanGestureRecognizer(target: self, 
                                                          action: #selector(handleEdgePan))
        leftEdgePan.edges = .left  // с левого края
        
        // 2. Настраиваем (опционально)
        leftEdgePan.maximumNumberOfTouches = 1
        
        // 3. Добавляем жест к главному view
        view.addGestureRecognizer(leftEdgePan)
        
        // 4. (Опционально) жест с правого края (открыть меню)
        let rightEdgePan = UIScreenEdgePanGestureRecognizer(target: self, 
                                                           action: #selector(handleEdgePan))
        rightEdgePan.edges = .right
        
        view.addGestureRecognizer(rightEdgePan)
    }
    
    @objc private func handleEdgePan(_ gesture: UIScreenEdgePanGestureRecognizer) {
        // Получаем текущую скорость и перевод (translation)
        let translation = gesture.translation(in: view)
        let velocity = gesture.velocity(in: view)
        
        switch gesture.state {
        case .began:
            print("Начало свайпа с края: \(gesture.edges)")
            
        case .changed:
            // Можно анимировать меню или transition в реальном времени
            let progress = abs(translation.x) / view.bounds.width
            print("Прогресс свайпа: \(progress), скорость: \(velocity.x)")
            
            // Пример: сдвигаем view или показываем preview меню
            // view.transform = CGAffineTransform(translationX: translation.x, y: 0)
            
        case .ended, .cancelled:
            // Решаем, завершать ли жест
            let progress = abs(translation.x) / view.bounds.width
            let velocityThreshold: CGFloat = 500  // пунктов в секунду
            
            if progress > 0.3 || abs(velocity.x) > velocityThreshold {
                // Завершаем жест (например, открыть меню или pop)
                if gesture.edges == .left {
                    // Свайп слева → назад
                    navigationController?.popViewController(animated: true)
                } else if gesture.edges == .right {
                    // Свайп справа → открыть боковое меню
                    print("Открываем боковое меню")
                }
            } else {
                // Откатываем анимацию
                UIView.animate(withDuration: 0.3) {
                    // view.transform = .identity
                }
            }
            
        default:
            break
        }
    }
}
```

### Лучшие практики UIScreenEdgePanGestureRecognizer в Swift 2026

- **edges** — обязательно указывай (.left, .right, .top, .bottom или комбинацию через [.left, .right])  
- **maximumNumberOfTouches = 1** — почти всегда (чтобы не мешал мультитач-жестам)  
- **velocity(in:)** — используй для определения, достаточно ли быстро пользователь свайпнул (threshold ~400–600)  
- **translation(in: view)** — считай прогресс как translation.x / view.bounds.width  
- **@objc** — обязательно для метода-обработчика  
- **@MainActor** — все обработчики жестов — на главном акторе  
- **Swift 6 strict concurrency** — UIGestureRecognizer полностью безопасен  
- **Документируйте** — пиши комментарий «UIScreenEdgePanGestureRecognizer — кастомный жест назад с левого края»

**Короткий девиз 2026**:
> UIScreenEdgePanGestureRecognizer — это когда тебе нужно распознавать **свайп именно с края экрана** (для навигации назад, открытия меню, переключения вкладок).  
> В 2026 году это **основной жест** для кастомной навигации и боковых панелей.  
> Всегда проверяй gesture.state, velocity и translation, чтобы жест был плавным и интуитивным.
