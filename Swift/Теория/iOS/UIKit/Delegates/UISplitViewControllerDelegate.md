**UISplitViewControllerDelegate** — это протокол в [[UIKit]], который позволяет тонко контролировать поведение **[[UISplitViewController]]** (Split View) — особенно в сценариях адаптивного интерфейса на iPad, iPhone Pro Max / Plus и в macOS Catalyst.

С помощью делегата вы можете:

- решать, сворачивать ли secondary в primary при переходе в компактный режим (collapse),
- определять, какой контроллер показывать при разворачивании (separate),
- выбирать, какой контроллер станет primary при изменении размера,
- настраивать кастомную анимацию перехода между состояниями.

Это один из ключевых инструментов для создания **адаптивного** интерфейса master-detail на iPad и больших iPhone.

### Актуальные методы делегата в 2026 году (iOS 18+)

| Метод делегата                                      | Когда вызывается                                                                 | Что возвращает / зачем нужен                                      | Самый частый сценарий в 2026 |
|-----------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------------------------------|------------------------------|
| `splitViewController(_:collapseSecondary:onto:)`    | Когда secondary нужно свернуть в primary (например, на iPhone или в compact width) | `Bool` — `true` = вы сами управляете collapse, `false` = система сама справится | Самый важный метод — предотвращение пустого primary |
| `splitViewController(_:separateSecondaryFrom:)`     | Когда secondary нужно отделить от primary (при расширении ширины)               | `UIViewController?` — контроллер, который станет secondary         | Возврат нужного detail-контроллера |
| `splitViewController(_:primaryViewControllerForCollapsing:)` | При сворачивании — какой контроллер станет primary                      | `UIViewController?` — контроллер primary                           | Редко, если нужно заменить primary |
| `splitViewController(_:primaryViewControllerForExpanding:)`  | При разворачивании — какой контроллер станет primary                    | `UIViewController?` — контроллер primary                           | Редко |
| `splitViewController(_:animationControllerForSeparating:from:)` | Кастомная анимация при разворачивании                                 | `UIViewControllerAnimatedTransitioning?`                           | Красивый transition между состояниями |
| `splitViewController(_:topColumnForCollapsingToProposedTopColumn:)` | iOS 14+ — какой столбец станет top при collapse                        | `UISplitViewController.Column`                                     | Редко, в triple-column режиме |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Адаптивный master-detail + обработка collapse)

```swift
class MainSplitViewController: UISplitViewController, UISplitViewControllerDelegate {
    
    private var masterVC: MasterViewController!
    private var detailVC: DetailViewController!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        preferredDisplayMode = .oneBesideSecondary  // iPad — две колонки
        delegate = self
        
        masterVC = MasterViewController()
        detailVC = DetailViewController()
        
        // На iPadOS 16+ можно использовать triple-column
        if #available(iOS 16.0, *) {
            preferredSplitBehavior = .overlay
            setViewController(masterVC, for: .primary)
            setViewController(detailVC, for: .secondary)
            setViewController(SupplementaryViewController(), for: .supplementary)
        } else {
            viewControllers = [masterVC, detailVC]
        }
    }
    
    // Самый важный метод — предотвращаем пустой экран на iPhone
    func splitViewController(
        _ svc: UISplitViewController,
        collapseSecondary secondaryViewController: UIViewController,
        onto primaryViewController: UIViewController
    ) -> Bool {
        // Если в detail ничего не выбрано → показываем placeholder или master
        if let detail = secondaryViewController as? DetailViewController,
           detail.currentItem == nil {
            
            // Можно заменить detail на что-то другое (например, welcome screen)
            return true  // ← мы сами обработали collapse
        }
        
        // Если есть выбранный элемент — оставляем detail поверх master
        return false  // система сама справится
    }
    
    // При разворачивании — восстанавливаем нужный detail
    func splitViewController(
        _ svc: UISplitViewController,
        separateSecondaryFrom primaryViewController: UIViewController
    ) -> UIViewController? {
        // Возвращаем сохранённый detail или создаём новый
        return detailVC
    }
    
    // Опционально — кастомная анимация разворачивания
    func splitViewController(
        _ svc: UISplitViewController,
        animationControllerForSeparating separatingViewController: UIViewController,
        from fromViewController: UIViewController
    ) -> UIViewControllerAnimatedTransitioning? {
        return SlideRightAnimator()
    }
}

// Простой аниматор (пример)
class SlideRightAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        0.35
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let toVC = transitionContext.viewController(forKey: .to) else { return }
        let container = transitionContext.containerView
        
        toVC.view.frame.origin.x = container.bounds.width
        container.addSubview(toVC.view)
        
        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: .curveEaseOut) {
            toVC.view.frame.origin.x = 0
        } completion: { _ in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
    }
}
```

### Современные альтернативы UISplitViewController в 2026 году

| Альтернатива                              | Когда лучше UISplitViewController                   | Минимальная версия | Плюсы                                |
| ----------------------------------------- | --------------------------------------------------- | ------------------ | ------------------------------------ |
| **NavigationSplitView** в SwiftUI         | Новый проект на [[SwiftUI]]                         | iOS 16+            | Декларативный стиль, проще адаптация |
| **UISplitViewController + triple-column** | [[UIKit]] + полноценный master-detail-supplementary | iOS 14+            | Нативная поддержка iPadOS            |
| **Custom UIView + two-column layout**     | Очень специфический дизайн                          | iOS 13+            | Полная кастомизация                  |
| **[[UITabBarController]] + detail view**  | Когда не нужна глубокая иерархия                    | iOS 13+            | Простота                             |

### Лучшие практики UISplitViewControllerDelegate в 2026 году

- **Всегда** реализуйте `collapseSecondary:onto:` — иначе на iPhone может остаться пустой экран  
- **Возвращайте** `true` в `collapseSecondary`, если сами обрабатываете сворачивание  
- **Используйте** `separateSecondaryFrom:` для возврата правильного detail-контроллера при разворачивании  
- **Для triple-column** (iPadOS 14+) — работайте с `.supplementary` и `.compact` режимами  
- **Для анимации** — реализуйте `animationControllerForSeparating` — красиво и профессионально  
- **Для SwiftUI** — используйте `NavigationSplitView` — UISplitViewControllerDelegate нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
// Предотвращаем пустой primary на iPhone при отсутствии выбранного элемента
func splitViewController(_ svc: UISplitViewController, collapseSecondary secondary: UIViewController, onto primary: UIViewController) -> Bool {
    // ...
}
```

**Короткий итог 2026**:
> `UISplitViewControllerDelegate` — протокол для управления **адаптивным поведением** колонок в Split View.  
> В 2026 году:  
> - ключевые методы — `collapseSecondary:onto:`, `separateSecondaryFrom:`  
> - самый важный — предотвращение пустого экрана на iPhone  
> - критично для iPad, iPhone Pro Max, Stage Manager, внешних дисплеев  
> - это **основа** создания профессионального master-detail интерфейса в UIKit  
