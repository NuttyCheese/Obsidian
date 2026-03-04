#uisplitviewcontroller #split-view #master-detail #uikit #ios #ipad #adaptive-ui #doublecolumn #triplecolumn #ios-14 #ios-16 #traitcollection #collapse #delegate

---
**(сплит-вью контроллер / контроллер разделённого экрана)**

**UISplitViewController** — это **контейнерный контроллер** в [[UIKit]], который управляет **разделённым (split) интерфейсом** с несколькими колонками (обычно 2 или 3), автоматически адаптируясь под размер экрана, ориентацию и устройство.

С iOS 14 (2020) он стал **одним из самых мощных и рекомендуемых** инструментов для создания **адаптивного master-detail интерфейса** на iPad, iPhone Pro Max / Plus и в macOS Catalyst / Stage Manager.

### 1. Основные режимы и колонки (2026 актуально)

| Режим / Стиль                             | Колонки                                      | Когда отображается                                   | Самый частый сценарий 2026 |
|-------------------------------------------|----------------------------------------------|------------------------------------------------------|-----------------------------|
| `.doubleColumn` (iOS 14+)                 | primary + secondary                          | iPad в landscape / iPhone в compact → secondary скрывается | Почти все современные приложения |
| `.tripleColumn` (iOS 14+)                 | primary + supplementary + secondary          | iPad в landscape / Stage Manager / macOS Catalyst    | Почта, Заметки, Файлы |
| `.compact` (автоматически на iPhone)      | Только primary (secondary collapse в primary) | iPhone portrait / compact width                      | Автоматическая адаптация |

**Ключевые колонки:**

- `.primary` — левая колонка (мастер / список / меню / навигация)
- `.supplementary` (iOS 14+) — средняя колонка (дополнительный контент, часто список подкатегорий)
- `.secondary` — правая колонка (детали / контент выбранного элемента)

### 2. Самый популярный и рекомендуемый паттерн 2026 года  
(Triple-column + NavigationController в каждой колонке + Delegate + Appearance)

```swift
import UIKit

class MainSplitViewController: UISplitViewController, UISplitViewControllerDelegate {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 1. Устанавливаем стиль (double или triple)
        preferredDisplayMode = .twoBesideSecondary  // две колонки рядом
        preferredSplitBehavior = .tile              // равномерное распределение
        
        // 2. Создаём контроллеры для колонок
        let primaryNav = UINavigationController(rootViewController: SidebarViewController())
        let supplementaryNav = UINavigationController(rootViewController: CategoriesViewController())
        let secondaryNav = UINavigationController(rootViewController: DetailPlaceholderViewController())
        
        // 3. Устанавливаем колонки
        setViewController(primaryNav, for: .primary)
        setViewController(supplementaryNav, for: .supplementary)
        setViewController(secondaryNav, for: .secondary)
        
        // 4. Делегат для управления collapse / expand
        delegate = self
        
        // 5. Настройка внешнего вида (опционально)
        setupAppearance()
    }
    
    private func setupAppearance() {
        let appearance = UINavigationBarAppearance()
        appearance.configureWithOpaqueBackground()
        appearance.backgroundColor = .systemBackground
        
        UINavigationBar.appearance().standardAppearance = appearance
        if #available(iOS 15.0, *) {
            UINavigationBar.appearance().scrollEdgeAppearance = appearance
        }
    }
    
    // MARK: - UISplitViewControllerDelegate
    
    func splitViewController(
        _ svc: UISplitViewController,
        collapseSecondary secondaryViewController: UIViewController,
        onto primaryViewController: UIViewController
    ) -> Bool {
        // На iPhone / compact width: если нет выбранного элемента → показываем placeholder
        if let detail = secondaryViewController as? UINavigationController,
           let topVC = detail.topViewController as? DetailViewController,
           topVC.item == nil {
            return true  // мы сами обрабатываем collapse
        }
        return false  // система сама справится
    }
    
    func splitViewController(
        _ svc: UISplitViewController,
        separateSecondaryFrom primaryViewController: UIViewController
    ) -> UIViewController? {
        // При разворачивании возвращаем сохранённый detail
        let detailNav = UINavigationController(rootViewController: DetailPlaceholderViewController())
        return detailNav
    }
    
    // Опционально: кастомная анимация разворачивания
    func splitViewController(
        _ svc: UISplitViewController,
        animationControllerForSeparating separatingViewController: UIViewController,
        from fromViewController: UIViewController
    ) -> UIViewControllerAnimatedTransitioning? {
        return SlideInAnimator()
    }
}

// Простой аниматор разворачивания (пример)
class SlideInAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        0.35
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let toVC = transitionContext.viewController(forKey: .to) else { return }
        let container = transitionContext.containerView
        
        toVC.view.frame.origin.x = container.bounds.width
        container.addSubview(toVC.view)
        
        UIView.animate(withDuration: transitionDuration(using: transitionContext),
                       delay: 0,
                       options: .curveEaseOut) {
            toVC.view.frame.origin.x = 0
        } completion: { _ in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
    }
}
```

### 3. Лучшие практики UISplitViewController в 2026 году

- **Стиль** — `.doubleColumn` для простых master-detail, `.tripleColumn` для сложных (как в Почте / Файлах)
- **preferredDisplayMode** — `.twoBesideSecondary` (две колонки рядом) — самый частый
- **preferredSplitBehavior** — `.tile` (равномерное) или `.overlay` (всплывающая supplementary)
- **Delegate** — обязательно реализуйте `collapseSecondary:onto:` для корректного поведения на iPhone
- **NavigationController в каждой колонке** — стандарт 2026 (push внутри primary / secondary)
- **traitCollectionDidChange** — адаптируйте ширину колонок, скрытие/показ элементов
- **showDetailViewController(_:sender:)** — удобный метод для обновления secondary колонки
- **Для [[SwiftUI]]** — используйте `NavigationSplitView` — UISplitViewController нужен только в [[UIKit]]
- **Доступность** — `accessibilityLabel` для колонок и элементов
- **Документируйте** — пишите комментарий:

```swift
/// Адаптивный сплит-вью контроллер с тремя колонками (iPad) и collapse на iPhone
class MainSplitViewController: UISplitViewController {
    // ...
}
```

**Короткий итог 2026**:
> **UISplitViewController** — **адаптивный контейнер** для **разделённого интерфейса** (master-detail / triple-column).  
> В 2026 году:  
> - ключевые свойства — `preferredDisplayMode`, `preferredSplitBehavior`, `setViewController(_:for:)`  
> - самый популярный паттерн — triple-column + NavigationController в каждой колонке + Delegate для collapse  
> - идеален для iPad, iPhone Pro Max, Stage Manager, macOS Catalyst  
> - в SwiftUI — заменяется на `NavigationSplitView`  
> - это **единственный нативный** способ создать профессиональный адаптивный интерфейс в UIKit  
