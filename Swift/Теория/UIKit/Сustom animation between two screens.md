**Кастомная анимация перехода между экранами** в [[iOS]] — это мощный инструмент, который позволяет полностью контролировать, как один экран «уходит», а другой «появляется».  
Самый распространённый и рекомендуемый способ в 2026 году — использование протокола **[[UIViewControllerAnimatedTransitioning]]** + **[[UIViewControllerTransitioningDelegate]]**.

Вот **полный, актуальный и максимально понятный** гид на 2026 год (Swift 6+, iOS 18+), как это правильно делать.

### 1. Что нужно реализовать (основные шаги)

1. Создать класс-аниматор, реализующий **`UIViewControllerAnimatedTransitioning`**  
2. (Опционально) создать класс для **интерактивного перехода** (свайп назад и т.д.) — `UIViewControllerInteractiveTransitioning`  
3. Реализовать **`UIViewControllerTransitioningDelegate`** в контроллере-источнике (откуда презентуем)  
4. Установить `transitioningDelegate` перед `present` или `push`  
5. (Опционально) добавить поддержку **dismiss** (обратный переход)

### 2. Самый популярный и современный пример 2026 года  
(Презентация с **fade + scale + slide** анимацией)

```swift
import UIKit

// MARK: - Аниматор (реализует UIViewControllerAnimatedTransitioning)
final class FadeScaleSlideAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    
    let duration: TimeInterval = 0.6
    let isPresenting: Bool  // true = present, false = dismiss
    
    init(isPresenting: Bool) {
        self.isPresenting = isPresenting
        super.init()
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return duration
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to),
              let fromView = fromVC.view,
              let toView = toVC.view else {
            transitionContext.completeTransition(false)
            return
        }
        
        let containerView = transitionContext.containerView
        
        // Начальные состояния
        if isPresenting {
            containerView.addSubview(toView)
            toView.alpha = 0
            toView.transform = CGAffineTransform(scaleX: 0.8, y: 0.8).translatedBy(x: 0, y: 50)
        } else {
            containerView.addSubview(toView)   // добавляем "предыдущий" экран под текущий
            containerView.addSubview(fromView)
        }
        
        UIView.animate(withDuration: duration, delay: 0, usingSpringWithDamping: 0.8, initialSpringVelocity: 0.5, options: .curveEaseOut) {
            if self.isPresenting {
                toView.alpha = 1
                toView.transform = .identity
            } else {
                fromView.alpha = 0
                fromView.transform = CGAffineTransform(scaleX: 0.8, y: 0.8).translatedBy(x: 0, y: -50)
            }
        } completion: { finished in
            // Обязательно завершаем переход!
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
    }
}

// MARK: - Делегат перехода (реализуем в контроллере-источнике)
extension YourFromViewController: UIViewControllerTransitioningDelegate {
    
    func animationController(forPresented presented: UIViewController,
                            presenting: UIViewController,
                            source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return FadeScaleSlideAnimator(isPresenting: true)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return FadeScaleSlideAnimator(isPresenting: false)
    }
}

// MARK: - Использование (перед present)
let toVC = YourToViewController()
toVC.transitioningDelegate = self   // self — это YourFromViewController
toVC.modalPresentationStyle = .fullScreen  // или .custom, .overFullScreen и т.д.
present(toVC, animated: true)
```

### Полезные улучшения и лучшие практики 2026

1. **Интерактивный переход** (свайп для отмены)

```swift
// Добавляем в делегат
func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
    return interactiveDismiss  // ваш UIPercentDrivenInteractiveTransition
}
```

2. **Комбинированный переход** (свой аниматор + системный navigation)

```swift
// В UINavigationControllerDelegate
func navigationController(_ navigationController: UINavigationController,
                         animationControllerFor operation: UINavigationController.Operation,
                         from fromVC: UIViewController,
                         to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    if operation == .push {
        return FadeScaleSlideAnimator(isPresenting: true)
    } else if operation == .pop {
        return FadeScaleSlideAnimator(isPresenting: false)
    }
    return nil
}
```

3. **Современные улучшения**
   - Используй **spring with damping** для естественной физики
   - Добавляй **delay** или **staggered animation** для нескольких элементов
   - Поддерживай **cancel** через `transitionContext.transitionWasCancelled`
   - Используй **snapshot** для сложных переходов (чтобы не ломать исходные вью)

### Короткий чек-лист «Что нужно сделать для кастомного перехода»

1. Создать класс, реализующий `UIViewControllerAnimatedTransitioning`  
2. Реализовать `transitionDuration` и `animateTransition`  
3. В контроллере-источнике реализовать `UIViewControllerTransitioningDelegate`  
4. Вернуть свой аниматор в методах делегата  
5. Установить `transitioningDelegate = self` перед `present` / `push`  
6. (Опционально) добавить интерактивность через `interactionControllerFor...`

**Короткий девиз 2026**:
> UIViewControllerAnimatedTransitioning — это когда ты хочешь **полностью контролировать анимацию** перехода между экранами: fade, slide, scale, bounce, 3D-эффекты и т.д.  
> В 2026 году это **основной** инструмент для кастомных переходов в [[UIKit]].  
> Для простых случаев — используй стандартные modalPresentationStyle или navigation transitions, для красивых — свой аниматор.
