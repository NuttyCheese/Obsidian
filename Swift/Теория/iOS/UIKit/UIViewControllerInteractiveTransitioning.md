**UIViewControllerInteractiveTransitioning** — это протокол в [[UIKit]], который позволяет сделать **модальный** или **навигационный** переход **интерактивным** (управляемым пользователем через жесты).

Он отвечает за **процентное управление** анимацией перехода (present / dismiss / push / pop) — то есть пользователь может:

- тянуть контроллер пальцем (swipe down to dismiss, drag from edge to pop)  
- отпускать — и анимация либо завершится, либо отменится в зависимости от прогресса  
- управлять скоростью и направлением анимации

Это основной механизм для создания **интерактивных жестовых переходов** (interactive transitions) — таких как:

- pull-down to dismiss (bottom sheet, card style)  
- edge swipe to go back (как в стандартной навигации iOS)  
- drag-to-dismiss модального экрана

### Ключевые методы протокола

| Метод                                            | Что возвращает / делает                                      | Когда вызывается                                      | Самый важный момент |
|--------------------------------------------------|---------------------------------------------------------------|--------------------------------------------------------|---------------------|
| `startInteractiveTransition(_:)`                 | `Void`                                                        | Когда пользователь начинает жест (например, начал тянуть) | Здесь сохраняем `transitionContext` |
| `updateInteractiveTransition(_:)`                | `Void`                                                        | При каждом изменении прогресса жеста (0.0...1.0)        | Обновляем UI в зависимости от прогресса |
| `finishInteractiveTransition()`                  | `Void`                                                        | Когда пользователь завершил жест (отпустил с прогрессом > threshold) | Завершаем анимацию до конца |
| `cancelInteractiveTransition()`                  | `Void`                                                        | Когда пользователь отпустил с прогрессом < threshold   | Откатываем анимацию обратно |
| `transitionDuration(using:)`                     | `TimeInterval`                                                | Определяет общую длительность анимации                 | Обычно 0.35–0.5 сек |
| `animateTransition(using:)`                      | `Void`                                                        | Вызывается, если переход не интерактивный              | Реализуем обычную анимацию |

### Самый популярный паттерн 2026 года  
(Интерактивный dismiss swipe-down для bottom sheet)

```swift
// 1. Аниматор (UIViewControllerAnimatedTransitioning)
class SlideDownAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    
    let isPresenting: Bool
    let duration: TimeInterval = 0.35
    
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
            UIView.animate(withDuration: duration, delay: 0, usingSpringWithDamping: 0.85, initialSpringVelocity: 0.5) {
                toVC.view.frame.origin.y = 0
            } completion: { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        } else {
            UIView.animate(withDuration: duration) {
                fromVC.view.frame.origin.y = container.bounds.height
            } completion: { _ in
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        }
    }
}

// 2. Интерактивный контроллер (UIViewControllerInteractiveTransitioning)
class InteractiveDismissTransition: UIPercentDrivenInteractiveTransition {
    
    var interactionInProgress = false
    var shouldComplete = false
    
    weak var transitionContext: UIViewControllerContextTransitioning?
    
    func startInteractiveTransition(_ transitionContext: UIViewControllerContextTransitioning) {
        super.startInteractiveTransition(transitionContext)
        self.transitionContext = transitionContext
        interactionInProgress = true
    }
    
    func update(_ percentComplete: CGFloat) {
        super.update(percentComplete)
        shouldComplete = percentComplete > 0.4  // порог завершения
    }
    
    func cancel() {
        interactionInProgress = false
        super.cancel()
    }
    
    func finish() {
        interactionInProgress = false
        super.finish()
    }
}

// 3. Переходный делегат (UIViewControllerTransitioningDelegate)
class CustomTransitionDelegate: NSObject, UIViewControllerTransitioningDelegate {
    
    let interactiveDismiss = InteractiveDismissTransition()
    
    func animationController(forPresented presented: UIViewController,
                             presenting: UIViewController,
                             source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        SlideDownAnimator(isPresenting: true)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        SlideDownAnimator(isPresenting: false)
    }
    
    func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        interactiveDismiss.interactionInProgress ? interactiveDismiss : nil
    }
}
```

### Использование в контроллере (swipe-down to dismiss)

```swift
class BottomSheetViewController: UIViewController {
    
    private let transitionDelegate = CustomTransitionDelegate()
    private let interactiveDismiss = transitionDelegate.interactiveDismiss
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        transitioningDelegate = transitionDelegate
        modalPresentationStyle = .custom
        
        // Добавляем жест pan для dismiss
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        view.addGestureRecognizer(panGesture)
    }
    
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        let velocity = gesture.velocity(in: view)
        
        let progress = max(0, min(1, translation.y / view.bounds.height))
        
        switch gesture.state {
        case .began:
            interactiveDismiss.interactionInProgress = true
            dismiss(animated: true)
            
        case .changed:
            interactiveDismiss.update(progress)
            
        case .ended, .cancelled:
            if progress > 0.4 || velocity.y > 1000 {
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
let bottomSheet = BottomSheetViewController()
bottomSheet.transitioningDelegate = transitionDelegate
present(bottomSheet, animated: true)
```

### Лучшие практики UIViewControllerInteractiveTransitioning в 2026 году

- **Всегда** реализуйте `UIPercentDrivenInteractiveTransition` (или наследника) — это самый простой и надёжный способ  
- **Порог завершения** — обычно 0.3–0.5 (чем выше, тем сложнее случайно закрыть)  
- **Учитывайте velocity** — если скорость свайпа большая, завершайте даже при малом прогрессе  
- **Для push/pop** — используйте `UINavigationControllerDelegate` + `interactionControllerFor`  
- **Для [[SwiftUI]]** — используйте `.interactiveDismissDisabled(false)` + `.transition` — интерактивные переходы в SwiftUI проще и мощнее  
- **Для доступности** — поддерживайте VoiceOver (accessibility) в жестах  
- **Документируйте** — пишите комментарий:

```swift
/// Интерактивный контроллер для swipe-down to dismiss bottom sheet
class InteractiveDismissTransition: UIPercentDrivenInteractiveTransition {
    // ...
}
```

**Короткий итог 2026**:
> `UIViewControllerInteractiveTransitioning` — протокол для **интерактивных (жестовых)** модальных и навигационных переходов.  
> В 2026 году:  
> - ключевой класс — `UIPercentDrivenInteractiveTransition`  
> - основные методы — `startInteractiveTransition`, `update`, `finish`, `cancel`  
> - идеален для bottom sheet, card dismiss, edge swipe back  
> - часто используется с `UIViewControllerTransitioningDelegate`  
> - это **единственный** способ сделать жестовый, управляемый пользователем переход в UIKit  
