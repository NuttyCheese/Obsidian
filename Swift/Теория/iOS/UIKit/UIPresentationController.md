**UIPresentationController** — это класс в [[UIKit]], который отвечает за **управление презентацией** (отображением) модального контроллера представления ([[UIViewController]]) на экране.

Он появился в iOS 8 (2014) вместе с введением **size classes** и адаптивных интерфейсов и остаётся **ключевым** инструментом для создания кастомных модальных презентаций в 2026 году.

### Основная роль UIPresentationController

UIPresentationController берёт на себя:

- определение **размера** и **позиции** модального контроллера на экране  
- управление **переходом** (transition) — анимацией появления/исчезновения  
- наложение **затемнения** (dimming view) или других оверлеев  
- обработку **адаптации** под разные size class (iPhone → iPad, Split View, Slide Over)  
- реакцию на **изменение черт окружения** (trait collection)  

Когда вы вызываете `present(_:animated:completion:)`, система автоматически создаёт `UIPresentationController` (или его подкласс), если не задан кастомный.

### Ключевые свойства и методы (самые важные в 2026)

| Свойство / Метод                                  | Тип / Значение по умолчанию  | Что делает / зачем нужно                         | Самый частый сценарий                              |
| ------------------------------------------------- | ---------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `presentedViewController`                         | `UIViewController`           | Контроллер, который мы презентуем                | Доступ к модальному VC                             |
| `presentingViewController`                        | `UIViewController?`          | Контроллер, от которого презентуем               | Возврат к presenting VC                            |
| `containerView`                                   | [[UIView]]                   | Контейнер, в который добавляется presented view  | Кастомный layout                                   |
| `frameOfPresentedViewInContainerView`             | [[CGRect]] (вычисляемое)     | Прямоугольник, где будет размещён presented view | **Самое важное** — здесь задаётся размер и позиция |
| `adaptivePresentationStyle(for:traitCollection:)` | [[UIModalPresentationStyle]] | Как адаптировать стиль на разных устройствах     | `.fullScreen` → `.pageSheet` на iPad               |
| `presentationTransitionWillBegin()`               | `Void`                       | Вызывается перед началом анимации появления      | Добавление затемнения                              |
| `presentationTransitionDidEnd(_:)`                | `Void`                       | Завершение анимации появления                    | Очистка, если отменили                             |
| `dismissalTransitionWillBegin()`                  | `Void`                       | Перед началом анимации ухода                     | Анимация затемнения                                |
| `dismissalTransitionDidEnd(_:)`                   | `Void`                       | После завершения ухода                           | Удаление оверлеев                                  |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Кастомный bottom sheet / card-style модалка)

```swift
class CustomPresentationController: UIPresentationController {
    
    private let dimmingView = UIView()
    
    override init(presentedViewController: UIViewController, presenting presentingViewController: UIViewController?) {
        super.init(presentedViewController: presentedViewController, presenting: presentingViewController)
        
        // Затемнение фона
        dimmingView.backgroundColor = UIColor.black.withAlphaComponent(0.4)
        dimmingView.alpha = 0
        dimmingView.translatesAutoresizingMaskIntoConstraints = false
        dimmingView.isUserInteractionEnabled = true
        
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(dismissPresented))
        dimmingView.addGestureRecognizer(tapGesture)
    }
    
    override var frameOfPresentedViewInContainerView: CGRect {
        // Кастомный размер: 80% высоты экрана, центрировано
        let containerBounds = containerView?.bounds ?? .zero
        let height = containerBounds.height * 0.8
        let width = containerBounds.width * 0.9
        
        return CGRect(
            x: (containerBounds.width - width) / 2,
            y: containerBounds.height - height - 40, // отступ снизу
            width: width,
            height: height
        )
    }
    
    override func presentationTransitionWillBegin() {
        guard let containerView else { return }
        
        containerView.addSubview(dimmingView)
        NSLayoutConstraint.activate([
            dimmingView.topAnchor.constraint(equalTo: containerView.topAnchor),
            dimmingView.bottomAnchor.constraint(equalTo: containerView.bottomAnchor),
            dimmingView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor),
            dimmingView.trailingAnchor.constraint(equalTo: containerView.trailingAnchor)
        ])
        
        // Анимация появления затемнения
        if let transitionCoordinator = presentedViewController.transitionCoordinator {
            transitionCoordinator.animate(alongsideTransition: { _ in
                self.dimmingView.alpha = 1
            })
        }
    }
    
    override func dismissalTransitionWillBegin() {
        // Анимация исчезновения затемнения
        if let transitionCoordinator = presentedViewController.transitionCoordinator {
            transitionCoordinator.animate(alongsideTransition: { _ in
                self.dimmingView.alpha = 0
            })
        }
    }
    
    override func containerViewWillLayoutSubviews() {
        presentedView?.frame = frameOfPresentedViewInContainerView
    }
    
    @objc private func dismissPresented() {
        presentedViewController.dismiss(animated: true)
    }
}
```

### Использование кастомного Presentation Controller

```swift
let modalVC = ModalContentViewController()
modalVC.modalPresentationStyle = .custom
modalVC.transitioningDelegate = self  // или отдельный объект

present(modalVC, animated: true)
```

```swift
extension ViewController: UIViewControllerTransitioningDelegate {
    func presentationController(forPresented presented: UIViewController,
                                presenting: UIViewController?,
                                source: UIViewController) -> UIPresentationController? {
        return CustomPresentationController(presentedViewController: presented, presenting: presenting)
    }
}
```

### Лучшие практики UIPresentationController в 2026 году

- **Всегда** переопределяйте `frameOfPresentedViewInContainerView` — это главный способ задать размер и позицию  
- **Для затемнения** — добавляйте `dimmingView` в `presentationTransitionWillBegin()` и анимируйте его alpha  
- **Для адаптации** — реализуйте `adaptivePresentationStyle(for:traitCollection:)`  
- **Для swipe-to-dismiss** — комбинируйте с `UIViewControllerTransitioningDelegate` и `UIPercentDrivenInteractiveTransition`  
- **Для [[SwiftUI]]** — используйте `.presentationDetents`, `.sheet` — `UIPresentationController` нужен только в [[UIKit]] или смешанных проектах  
- **Для iPad / Stage Manager** — проверяйте `traitCollection.horizontalSizeClass` и адаптируйте размер  
- **Документируйте** — пишите комментарий:

```swift
/// Кастомный UIPresentationController для bottom sheet с затемнением и swipe-to-dismiss
class CustomPresentationController: UIPresentationController { ... }
```

**Короткий итог 2026**:
> `UIPresentationController` — это контроллер, который управляет **позицией**, **размером** и **переходом** модального контроллера.  
> В 2026 году:  
> - ключевой метод — `frameOfPresentedViewInContainerView` (задаёт размер и позицию)  
> - идеален для bottom sheet, card-style, custom modal, popover  
> - часто используется вместе с [[UIViewControllerTransitioningDelegate]]  
> - это **основа** создания нестандартных модальных презентаций в UIKit  
