**UINavigationControllerDelegate** — это протокол в [[UIKit]], который позволяет **тонко контролировать поведение** навигационного контроллера ([[UINavigationController]]), особенно:

- **кастомизировать анимацию** push / pop переходов между экранами  
- делать **интерактивные жестовые переходы** (swipe from edge to go back)  
- реагировать на события добавления/удаления контроллеров в стек  
- управлять отображением navigation bar, toolbar и т.д.

Это **один из самых важных** делегатов для создания плавных, современных и кастомных навигационных переходов в UIKit-приложениях 2026 года.

### Основные методы протокола (самые используемые в 2026)

| Метод делегата                                      | Когда вызывается                                                                 | Что возвращает / зачем нужен                                      | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------------------------------|-----------------------|
| `navigationController(_:animationControllerFor:from:to:)` | Перед push / pop — определяет аниматор перехода                                 | `UIViewControllerAnimatedTransitioning?` — кастомный аниматор     | Кастомная анимация (zoom, slide, fade) |
| `navigationController(_:interactionControllerFor:)` | Когда переход должен быть интерактивным (жест)                                  | `UIViewControllerInteractiveTransitioning?` — контроллер жеста     | Swipe from left edge to pop |
| `navigationController(_:willShow:animated:)`        | Перед показом нового контроллера                                                | `Void` — подготовка UI (обновление кнопок, title)                 | Скрытие/показ navigation bar |
| `navigationController(_:didShow:animated:)`         | После показа контроллера                                                        | `Void` — реакция на завершение перехода                           | Логирование, аналитика |
| `navigationControllerSupportedInterfaceOrientations(_:)` | Запрос поддерживаемых ориентаций экрана                                       | `UIInterfaceOrientationMask`                                       | Ограничение ориентации на отдельных экранах |
| `navigationControllerPreferredInterfaceOrientationForPresentation(_:)` | Предпочитаемая ориентация при презентации                                   | `UIInterfaceOrientation`                                           | Редко, для модальных экранов |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Кастомный push/pop + интерактивный edge swipe back)

```swift
class CustomNavigationDelegate: NSObject, UINavigationControllerDelegate {
    
    // Кастомная анимация push / pop
    func navigationController(_ navigationController: UINavigationController,
                              animationControllerFor operation: UINavigationController.Operation,
                              from fromVC: UIViewController,
                              to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        
        switch operation {
        case .push:
            return PushAnimator()
        case .pop:
            return PopAnimator()
        default:
            return nil
        }
    }
    
    // Интерактивный контроллер для pop (swipe from left edge)
    func navigationController(_ navigationController: UINavigationController,
                              interactionControllerFor animationController: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        
        // Возвращаем интерактивный контроллер только если жест начат
        return interactivePopTransition?.hasStarted == true ? interactivePopTransition : nil
    }
}

// Аниматор для push
class PushAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval { 0.35 }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to) else { return }
        
        let container = transitionContext.containerView
        container.addSubview(toVC.view)
        
        toVC.view.frame.origin.x = container.bounds.width
        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: .curveEaseOut) {
            toVC.view.frame.origin.x = 0
        } completion: { _ in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
    }
}

// Аниматор для pop
class PopAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval { 0.35 }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to) else { return }
        
        let container = transitionContext.containerView
        container.addSubview(toVC.view)
        container.addSubview(fromVC.view)
        
        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: .curveEaseIn) {
            fromVC.view.frame.origin.x = container.bounds.width
        } completion: { _ in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
    }
}

// Интерактивный контроллер (swipe from left)
class InteractivePopTransition: UIPercentDrivenInteractiveTransition {
    var hasStarted = false
    var shouldFinish = false
    
    override func update(_ percentComplete: CGFloat) {
        super.update(percentComplete)
        shouldFinish = percentComplete > 0.4
    }
}
```

### Использование в контроллере

```swift
class MainNavigationController: UINavigationController {
    
    private let customDelegate = CustomNavigationDelegate()
    private let interactivePop = InteractivePopTransition()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        delegate = customDelegate
        
        // Добавляем системный edge swipe (если нужно кастомный — отключаем)
        interactivePopGestureRecognizer?.isEnabled = false
        
        // Наш кастомный жест (опционально)
        let edgePan = UIScreenEdgePanGestureRecognizer(target: self, action: #selector(handleEdgePan(_:)))
        edgePan.edges = .left
        view.addGestureRecognizer(edgePan)
    }
    
    @objc private func handleEdgePan(_ gesture: UIScreenEdgePanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        let progress = translation.x / view.bounds.width
        
        switch gesture.state {
        case .began:
            interactivePop.hasStarted = true
            popViewController(animated: true)
            
        case .changed:
            interactivePop.update(progress)
            
        case .ended, .cancelled:
            if progress > 0.4 || gesture.velocity(in: view).x > 800 {
                interactivePop.finish()
            } else {
                interactivePop.cancel()
            }
            
        default:
            break
        }
    }
}
```

### Лучшие практики UINavigationControllerDelegate в 2026 году

- **Всегда** реализуйте оба метода: `animationControllerFor` и `interactionControllerFor`  
- **Для push/pop** — возвращайте разные аниматоры в зависимости от `operation`  
- **Для интерактивности** — используйте [[UIPercentDrivenInteractiveTransition]] (или наследника)  
- **Порог завершения** — обычно 0.3–0.5 (чем выше, тем сложнее случайно вернуться назад)  
- **Velocity** — если скорость жеста большая — завершайте даже при малом прогрессе  
- **Для [[SwiftUI]]** — используйте `.navigationTransition(.slide)` + `.gesture(DragGesture())` — делегат нужен только в UIKit  
- **Для доступности** — поддерживайте VoiceOver и системные жесты  
- **Документируйте** — пишите комментарий:

```swift
/// Делегат навигации с кастомной анимацией push/pop и интерактивным swipe-back
class CustomNavigationDelegate: NSObject, UINavigationControllerDelegate { ... }
```

**Короткий итог 2026**:
> `UINavigationControllerDelegate` — протокол для **кастомизации переходов** в навигационном стеке (push / pop).  
> В 2026 году:  
> - ключевые методы — `animationControllerFor`, `interactionControllerFor`  
> - самый популярный сценарий — кастомный slide/zoom + swipe from edge to pop  
> - часто используется вместе с `UIPercentDrivenInteractiveTransition`  
> - это **единственный** способ сделать красивые и интерактивные навигационные переходы в UIKit  
