**UIPercentDrivenInteractiveTransition** — это **готовый класс** в [[UIKit]], который реализует протокол [[UIViewControllerInteractiveTransitioning]] и предназначен для создания **интерактивных (управляемых жестом) переходов**.

Он берёт на себя почти всю работу по управлению прогрессом анимации:

- отслеживание прогресса жеста (от 0.0 до 1.0),
- обновление анимации в реальном времени,
- завершение или отмена перехода в зависимости от того, насколько далеко пользователь «протянул»,
- обработка velocity (скорости жеста),
- автоматическая поддержка spring-анимации и timing curves.

Это **самый популярный и рекомендуемый** способ сделать swipe-to-dismiss, edge-swipe-back, drag-to-close и любые другие жестовые переходы в UIKit в 2026 году.

### Когда использовать UIPercentDrivenInteractiveTransition

| Сценарий                                            | Почему именно этот класс                     | Альтернатива (когда не использовать)                           |
| --------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------- |
| Swipe down to dismiss bottom sheet / card           | Очень удобно + встроенная поддержка velocity | Кастомный интерактивный контроллер                             |
| Edge swipe to go back (как в стандартной навигации) | Почти идентично системному поведению         | [[UINavigationControllerDelegate]] + встроенный gesture        |
| Pull-to-close модального экрана                     | Простая интеграция с pan gesture             | —                                                              |
| Interactive present (drag from bottom)              | Можно, но реже используется                  | Кастомный контроллер                                           |
| Любая анимация, где прогресс зависит от жеста       | Самый быстрый и надёжный способ              | Полностью кастомный `UIViewControllerInteractiveTransitioning` |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Интерактивный dismiss swipe-down для bottom sheet)

```swift
// 1. Аниматор (UIViewControllerAnimatedTransitioning)
class SlideUpAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    
    let isPresenting: Bool
    let duration: TimeInterval = 0.4
    
    init(isPresenting: Bool) {
        self.isPresenting = isPresenting
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        duration
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to) else { return }
        
        let container = transitionContext.containerView
        
        if isPresenting {
            container.addSubview(toVC.view)
            toVC.view.frame.origin.y = container.bounds.height
            UIView.animate(withDuration: duration, delay: 0, usingSpringWithDamping: 0.85, initialSpringVelocity: 0.6) {
                toVC.view.frame.origin.y = 0
            } completion: { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        } else {
            UIView.animate(withDuration: duration, delay: 0, options: .curveEaseOut) {
                fromVC.view.frame.origin.y = container.bounds.height
            } completion: { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        }
    }
}

// 2. Интерактивный контроллер — просто наследуемся от готового класса
class InteractiveSwipeDismiss: UIPercentDrivenInteractiveTransition {
    
    var hasStarted = false
    var shouldFinish = false
    
    weak var transitionContext: UIViewControllerContextTransitioning?
    
    override func startInteractiveTransition(_ transitionContext: UIViewControllerContextTransitioning) {
        super.startInteractiveTransition(transitionContext)
        self.transitionContext = transitionContext
        hasStarted = true
    }
    
    override func update(_ percentComplete: CGFloat) {
        super.update(percentComplete)
        // Порог завершения — можно настраивать
        shouldFinish = percentComplete > 0.35 || percentComplete > 0.5
    }
    
    override func finish() {
        hasStarted = false
        super.finish()
    }
    
    override func cancel() {
        hasStarted = false
        super.cancel()
    }
}

// 3. Переходный делегат (UIViewControllerTransitioningDelegate)
class SheetTransitionDelegate: NSObject, UIViewControllerTransitioningDelegate {
    
    let interactiveDismiss = InteractiveSwipeDismiss()
    
    func animationController(forPresented presented: UIViewController,
                             presenting: UIViewController,
                             source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        SlideUpAnimator(isPresenting: true)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        SlideUpAnimator(isPresenting: false)
    }
    
    func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        interactiveDismiss.hasStarted ? interactiveDismiss : nil
    }
}
```

### Использование в контроллере (swipe-down to dismiss)

```swift
class ResizableSheetViewController: UIViewController {
    
    private let transitionDelegate = SheetTransitionDelegate()
    private let interactiveDismiss = transitionDelegate.interactiveDismiss
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Настройка модальной презентации
        modalPresentationStyle = .custom
        transitioningDelegate = transitionDelegate
        
        // Жест для swipe-down
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePanDismiss(_:)))
        view.addGestureRecognizer(pan)
    }
    
    @objc private func handlePanDismiss(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        let velocity = gesture.velocity(in: view)
        
        // Прогресс = насколько далеко вниз протянули
        var progress = translation.y / view.bounds.height
        progress = max(0.0, min(1.0, progress))
        
        switch gesture.state {
        case .began:
            interactiveDismiss.hasStarted = true
            dismiss(animated: true)
            
        case .changed:
            interactiveDismiss.update(progress)
            
        case .ended, .cancelled:
            // Учитываем скорость жеста
            if progress > 0.4 || velocity.y > 800 {
                interactiveDismiss.finish()
            } else {
                interactiveDismiss.cancel()
            }
            
        default:
            break
        }
    }
}

// Показываем контроллер
let sheet = ResizableSheetViewController()
sheet.transitioningDelegate = transitionDelegate
present(sheet, animated: true)
```

### Лучшие практики UIPercentDrivenInteractiveTransition в 2026 году

- **Используйте именно этот класс** — он уже решает 90% задач (progress, velocity, spring, completion)  
- **Порог завершения** — обычно 0.3–0.5 (чем выше, тем сложнее случайно закрыть)  
- **Velocity** — если скорость жеста > 600–1000 pt/s — завершайте даже при малом прогрессе  
- **Для push/pop** — используйте `UINavigationControllerDelegate` + `interactionControllerFor`  
- **Для [[SwiftUI]]** — `.interactiveDismissDisabled(false)` + `.transition` + `.gesture(DragGesture())` — в SwiftUI проще  
- **Для доступности** — поддерживайте VoiceOver и жесты Accessibility  
- **Документируйте** — пишите комментарий:

```swift
/// Интерактивный контроллер для swipe-down to dismiss модального листа
class InteractiveSwipeDismiss: UIPercentDrivenInteractiveTransition {
    // ...
}
```

**Короткий итог 2026**:
> `UIPercentDrivenInteractiveTransition` — **готовый класс** для создания **интерактивных жестовых переходов** (swipe to dismiss, edge pop и т.д.).  
> В 2026 году:  
> - ключевые методы — `update(_:)`, `finish()`, `cancel()`  
> - самый популярный сценарий — pull-down to close bottom sheet  
> - почти всегда используется вместе с `UIViewControllerTransitioningDelegate`  
> - это **самый простой и надёжный** способ сделать управляемый пользователем переход в UIKit  
