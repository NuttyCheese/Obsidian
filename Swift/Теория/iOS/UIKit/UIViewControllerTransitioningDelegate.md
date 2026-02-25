**UIViewControllerTransitioningDelegate** — это протокол в [[UIKit]], который позволяет **полностью кастомизировать** анимацию перехода между двумя контроллерами представления (present / dismiss).

Он отвечает за:

- предоставление **аниматора** (animation controller) для презентации и отмены презентации  
- предоставление **интерактивного аниматора** (interaction controller) — для управления переходом жестами (swipe to dismiss, drag to close и т.д.)  
- определение **стилей презентации** (presentation style) — fullScreen, overCurrentContext, custom и т.д.

Это основной инструмент для создания **нестандартных модальных переходов** в [[iOS]]-приложениях.

### Когда использовать UIViewControllerTransitioningDelegate (актуально 2026)

| Сценарий                                    | Почему именно transitioning delegate                               | Альтернатива                                 |
| ------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------- |
| Кастомная анимация модального экрана        | Стандартный crossDissolve / coverVertical выглядит скучно          | —                                            |
| Bottom sheet / half-modal / card-style      | Нужен контроллер с кастомным размером и анимацией снизу            | [[UISheetPresentationController]] (iOS 15+)  |
| Full-screen анимация с зумом / slide / fade | Хотите сложную анимацию (scale, rotate, interactive)               | —                                            |
| Интерактивный dismiss (swipe down to close) | Пользователь может тянуть контроллер вниз пальцем                  | [[UIViewControllerInteractiveTransitioning]] |
| Переход с gesture-driven анимацией          | Drag-to-dismiss, pan-to-present                                    | [[UIPercentDrivenInteractiveTransition]]     |
| Кастомный presentation style                | overCurrentContext, custom, popover и т.д. с собственной анимацией | —                                            |

### Основные методы протокола (самые важные)

| Метод делегата                                      | Что возвращает                                       | Когда вызывается                                      | Самый частый сценарий |
|-----------------------------------------------------|------------------------------------------------------|--------------------------------------------------------|-----------------------|
| `animationController(forPresented:presenting:source:)` | `UIViewControllerAnimatedTransitioning?`             | Перед презентацией (present)                           | Кастомная анимация появления |
| `animationController(forDismissed:)`                | `UIViewControllerAnimatedTransitioning?`             | Перед dismiss                                          | Кастомная анимация ухода |
| `interactionControllerForPresentation(using:)`     | `UIViewControllerInteractiveTransitioning?`          | Если презентация должна быть интерактивной             | Редко (чаще для dismiss) |
| `interactionControllerForDismissal(using:)`        | `UIViewControllerInteractiveTransitioning?`          | Если dismiss должен быть интерактивным (swipe)         | Swipe-down to dismiss |
| `presentationController(forPresented:presenting:source:)` | `UIPresentationController?`                   | Для кастомного presentation style                      | Bottom sheet, custom frame |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Интерактивный bottom sheet с кастомной анимацией)

```swift
class CustomTransitionDelegate: NSObject, UIViewControllerTransitioningDelegate {
    
    // Аниматор для презентации
    func animationController(forPresented presented: UIViewController,
                             presenting: UIViewController,
                             source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return SlideUpAnimator(isPresenting: true)
    }
    
    // Аниматор для dismiss
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return SlideUpAnimator(isPresenting: false)
    }
    
    // Интерактивный dismiss (swipe down)
    func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        return interactiveDismissController  // UIPercentDrivenInteractiveTransition или кастомный
    }
}

// Аниматор (простой slide up/down)
class SlideUpAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    
    let isPresenting: Bool
    let duration: TimeInterval = 0.35
    
    init(isPresenting: Bool) {
        self.isPresenting = isPresenting
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return duration
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to) else { return }
        
        let containerView = transitionContext.containerView
        
        if isPresenting {
            containerView.addSubview(toVC.view)
            toVC.view.frame.origin.y = containerView.bounds.height
            UIView.animate(withDuration: duration, delay: 0, usingSpringWithDamping: 0.8, initialSpringVelocity: 0.5) {
                toVC.view.frame.origin.y = 0
            } completion: { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        } else {
            UIView.animate(withDuration: duration, animations: {
                fromVC.view.frame.origin.y = containerView.bounds.height
            }) { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        }
    }
}

// Интерактивный контроллер (swipe down to dismiss)
class InteractiveDismissController: UIPercentDrivenInteractiveTransition {
    var interactionInProgress = false
    var shouldComplete = false
    
    override func update(_ percentComplete: CGFloat) {
        super.update(percentComplete)
        shouldComplete = percentComplete > 0.5
    }
    
    override func cancel() {
        interactionInProgress = false
        super.cancel()
    }
    
    override func finish() {
        interactionInProgress = false
        super.finish()
    }
}
```

### Использование в контроллере

```swift
class BottomSheetViewController: UIViewController {
    
    private let transitionDelegate = CustomTransitionDelegate()
    private let interactiveController = InteractiveDismissController()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Настраиваем интерактивный dismiss
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        view.addGestureRecognizer(panGesture)
    }
    
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        
        switch gesture.state {
        case .began:
            interactiveController.interactionInProgress = true
            dismiss(animated: true)
            
        case .changed:
            let progress = max(0, min(1, translation.y / view.bounds.height))
            interactiveController.update(progress)
            
        case .ended, .cancelled:
            if interactiveController.shouldComplete || translation.y > 200 {
                interactiveController.finish()
            } else {
                interactiveController.cancel()
            }
            
        default:
            break
        }
    }
}

// Показываем контроллер
let bottomSheet = BottomSheetViewController()
bottomSheet.modalPresentationStyle = .custom
bottomSheet.transitioningDelegate = transitionDelegate
present(bottomSheet, animated: true)
```

### Лучшие практики UIViewControllerTransitioningDelegate в 2026

- **Всегда** реализуйте оба аниматора: для present и dismiss  
- **Для интерактивного dismiss** — используйте [[UIPercentDrivenInteractiveTransition]] или кастомный  
- **Для bottom sheet** — часто используют `.pageSheet` + `UISheetPresentationController` (iOS 15+), но для полной кастомизации — transitioning delegate  
- **Для производительности** — держите анимацию простой (spring, easeInOut), избегайте тяжёлых view в transition  
- **Для SwiftUI** — используйте `.transition`, `.matchedGeometryEffect`, `.presentationDetents` — transitioning delegate нужен только в UIKit или смешанных проектах  
- **Документируйте** — пишите комментарий:

```swift
/// Кастомный transitioning delegate для slide-up bottom sheet с интерактивным dismiss
class CustomTransitionDelegate: NSObject, UIViewControllerTransitioningDelegate { ... }
```

**Короткий итог 2026**:
> `UIViewControllerTransitioningDelegate` — протокол для **полной кастомизации** модальных переходов (present / dismiss).  
> В 2026 году:  
> - ключевые методы — `animationController(forPresented:)`, `animationController(forDismissed:)`, `interactionControllerForDismissal`  
> - идеален для bottom sheet, card-style, zoom, interactive swipe  
> - часто комбинируется с `UIPercentDrivenInteractiveTransition`  
> - это **единственный** способ сделать действительно кастомную анимацию модального экрана в UIKit  
