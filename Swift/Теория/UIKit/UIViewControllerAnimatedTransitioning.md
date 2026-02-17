`UIViewControllerAnimatedTransitioning` — протокол, который **описывает анимацию перехода между двумя [[UIViewController]]**.

Важно:

> Он **НЕ запускает переход**,  
> он **НЕ решает, когда переход нужен**,  
> он **ТОЛЬКО рисует анимацию**.

[[UIKit]] вызывает твой объект, когда **решено**:

- показать новый экран
    
- скрыть старый
    
- анимировать это нестандартно
    

---

## 2. Где он живёт в системе переходов

Полная цепочка:

```
UIViewController
 └─ transitioningDelegate
       ├─ animationController(forPresented:)
       └─ animationController(forDismissed:)
            ↓
 UIViewControllerAnimatedTransitioning
```

Для `UINavigationController`:

```
UINavigationControllerDelegate
 └─ animationControllerForOperation
```

---

## 3. Минимальный протокол

```swift
protocol UIViewControllerAnimatedTransitioning {
    func transitionDuration(
        using transitionContext: UIViewControllerContextTransitioning?
    ) -> TimeInterval

    func animateTransition(
        using transitionContext: UIViewControllerContextTransitioning
    )
}
```

### Обязанности:

- сказать **длительность**
    
- выполнить **анимацию**
    
- **сообщить о завершении**
    

---

## 4. Что такое `transitionContext`

`UIViewControllerContextTransitioning` — это **контейнер перехода**.

Он даёт:

- `viewController(forKey:)`
    
- `view(forKey:)`
    
- `containerView`
    
- информацию об интерактивности
    
- статус отмены
    

---

## 5. Базовый пример: fade transition

### Аниматор

```swift
final class FadeAnimator: NSObject, UIViewControllerAnimatedTransitioning {

    func transitionDuration(
        using transitionContext: UIViewControllerContextTransitioning?
    ) -> TimeInterval {
        return 0.3
    }

    func animateTransition(
        using transitionContext: UIViewControllerContextTransitioning
    ) {
        guard
            let toView = transitionContext.view(forKey: .to)
        else { return }

        let container = transitionContext.containerView
        toView.alpha = 0
        container.addSubview(toView)

        UIView.animate(
            withDuration: transitionDuration(using: transitionContext),
            animations: {
                toView.alpha = 1
            },
            completion: { finished in
                transitionContext.completeTransition(
                    !transitionContext.transitionWasCancelled
                )
            }
        )
    }
}
```

---

## 6. Почему `containerView` важен

❌ Ошибка новичков:

```swift
fromVC.view.addSubview(toVC.view)
```

✅ Правильно:

```swift
transitionContext.containerView.addSubview(toView)
```

UIKit:

- управляет иерархией
    
- следит за жестами
    
- может отменить переход
    

---

## 7. `.from` и `.to`

```swift
let fromVC = transitionContext.viewController(forKey: .from)
let toVC = transitionContext.viewController(forKey: .to)
```

Но:

- **используй `view(forKey:)`**, а не `.view`
    
- UIKit может возвращать snapshot
    

---

## 8. Пример: slide transition

```swift
final class SlideAnimator: NSObject, UIViewControllerAnimatedTransitioning {

    func transitionDuration(
        using transitionContext: UIViewControllerContextTransitioning?
    ) -> TimeInterval {
        0.35
    }

    func animateTransition(
        using transitionContext: UIViewControllerContextTransitioning
    ) {
        guard
            let fromView = transitionContext.view(forKey: .from),
            let toView = transitionContext.view(forKey: .to)
        else { return }

        let container = transitionContext.containerView
        let width = container.bounds.width

        toView.frame = container.bounds.offsetBy(dx: width, dy: 0)
        container.addSubview(toView)

        UIView.animate(
            withDuration: transitionDuration(using: transitionContext),
            animations: {
                fromView.frame = fromView.frame.offsetBy(dx: -width / 3, dy: 0)
                toView.frame = container.bounds
            },
            completion: { _ in
                transitionContext.completeTransition(
                    !transitionContext.transitionWasCancelled
                )
            }
        )
    }
}
```

---

## 9. Present vs Dismiss

Один и тот же аниматор **может вести себя по-разному**:

```swift
final class ModalAnimator: NSObject, UIViewControllerAnimatedTransitioning {

    let isPresenting: Bool

    init(isPresenting: Bool) {
        self.isPresenting = isPresenting
    }

    func animateTransition(using ctx: UIViewControllerContextTransitioning) {
        isPresenting
        ? animatePresentation(ctx)
        : animateDismissal(ctx)
    }
}
```

---

## 10. Связь с transitioningDelegate

```swift
final class ModalVC: UIViewController {

    private let animator = FadeAnimator()

    override func viewDidLoad() {
        super.viewDidLoad()
        transitioningDelegate = self
        modalPresentationStyle = .custom
    }
}

extension ModalVC: UIViewControllerTransitioningDelegate {
    func animationController(
        forPresented presented: UIViewController,
        presenting: UIViewController,
        source: UIViewController
    ) -> UIViewControllerAnimatedTransitioning? {
        FadeAnimator()
    }

    func animationController(
        forDismissed dismissed: UIViewController
    ) -> UIViewControllerAnimatedTransitioning? {
        FadeAnimator()
    }
}
```

---

## 11. Интерактивные переходы (важно)

`UIViewControllerAnimatedTransitioning` **не знает про жесты**.

Для интерактивности нужен:

- `UIPercentDrivenInteractiveTransition`
    
- `UIViewControllerInteractiveTransitioning`
    

Пример:

- свайп для dismiss
    
- процент прогресса
    

---

## 12. transitionWasCancelled

Всегда учитывай:

```swift
transitionContext.transitionWasCancelled
```

Если не учесть:

- экран может остаться в иерархии
    
- будут баги с жестами
    

---

## 13. Snapshot-based анимации

Часто вместо `view` используют:

```swift
let snapshot = fromView.snapshotView(afterScreenUpdates: false)
```

Плюсы:

- производительность
    
- нет конфликтов с layout
    

---

## 14. Типичные ошибки

❌ Не вызвать `completeTransition`  
❌ Работать с `.view` напрямую  
❌ Игнорировать cancel  
❌ Менять rootViewController  
❌ Делать логику внутри аниматора

Аниматор — **тупой художник**, не архитектор.

---

## 15. Когда использовать

✅ Кастомные модальные экраны  
✅ Интерактивные dismiss  
✅ Сложные переходы  
❌ Обычные push/pop  
❌ Бизнес-логика

UIKit transitions — мощный, но тяжёлый инструмент.

---

## 16. Связь с прошлым и будущим

Прошлое:

- Core Animation напрямую
    
- [[Swift/Теория/UIKit/UIView]] animations
    

Настоящее:

- `UIViewControllerAnimatedTransitioning`
    

Будущее:

- UIKit будет жить
    
- SwiftUI решает иначе, но UIKit никуда не делся
    

---

## 17. Короткий итог

- это протокол анимации
    
- UIKit управляет жизненным циклом
    
- ты рисуешь переход
    
- контекст — твой [[API]]
    
- `completeTransition` обязателен
    

---
