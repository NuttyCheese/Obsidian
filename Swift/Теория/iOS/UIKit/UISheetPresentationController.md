**UISheetPresentationController** — это контроллер презентации, представленный в **iOS 15** (2021), который позволяет показывать модальные контроллеры в виде **листа** (sheet), выезжающего снизу экрана с возможностью изменения высоты, свайпа для закрытия и адаптивного поведения.

Это **самый популярный и рекомендуемый** способ реализации современных **bottom sheet**, **half sheet**, **card-style** и **resizable modal** экранов в [[UIKit]]-приложениях 2026 года.

### Основные возможности UISheetPresentationController

| Свойство / Метод                                   | Тип / Значение по умолчанию                        | Что делает / зачем нужно                                     | Самый частый сценарий                           |
| -------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| `detents`                                          | `[UISheetPresentationController.Detent]`           | Массив высот, между которыми может "застревать" лист         | `.medium()`, `.large()` — самые частые          |
| `selectedDetentIdentifier`                         | `UISheetPresentationController.Detent.Identifier?` | Текущая выбранная высота (можно менять программно)           | Переключение между medium и large               |
| `prefersGrabberVisible`                            | `Bool` (false)                                     | Показывать ли "полоску-захват" сверху листа                  | Почти всегда `true`                             |
| `prefersScrollingExpandsWhenScrolledToEdge`        | `Bool` (true)                                      | Расширять лист при скролле до верха контента                 | `false` — если не хотите авто-расширения        |
| `largestUndimmedDetentIdentifier`                  | `UISheetPresentationController.Detent.Identifier?` | До какой высоты фон не затемняется                           | `.medium` — затемнение только при полном экране |
| `prefersEdgeAttachedInCompactHeight`               | `Bool` (true)                                      | Прикреплять лист к нижнему краю в компактной высоте          | Обычно `true`                                   |
| `widthFollowsPreferredContentSizeWhenEdgeAttached` | `Bool` (true)                                      | Ширина листа следует `preferredContentSize` при прикреплении | Редко отключают                                 |
| `prefersDimmedScrimVisible` (iOS 16+)              | [[Bool]] (true)                                    | Показывать ли затемнение при полном экране                   | Обычно `true`                                   |
| `preferredCornerRadius` (iOS 16+)                  | [[CGFloat]]`?`                                     | Скругление углов листа                                       | 16–24 pt — стандарт                             |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Resizable bottom sheet + UISheetPresentationController + [[Swift Concurrency]])

```swift
import UIKit

class BottomSheetViewController: UIViewController {
    
    // Контент, который будет внутри листа
    private let contentView = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Настройка sheet presentation (самое важное)
        if let sheet = sheetPresentationController {
            sheet.detents = [.medium(), .large()]                    // две высоты
            sheet.selectedDetentIdentifier = .medium                 // стартуем с medium
            sheet.prefersGrabberVisible = true                       // полоска сверху
            sheet.prefersScrollingExpandsWhenScrolledToEdge = false // не расширять при скролле
            sheet.largestUndimmedDetentIdentifier = .large           // затемнение только в full screen
            sheet.prefersEdgeAttachedInCompactHeight = true          // прикрепление к краю
            sheet.preferredCornerRadius = 24                         // красивое скругление
            sheet.prefersDimmedScrimVisible = true                   // затемнение фона
        }
        
        // Добавляем контент (пример)
        let label = UILabel()
        label.text = "Это bottom sheet\nс двумя высотами"
        label.numberOfLines = 0
        label.textAlignment = .center
        label.translatesAutoresizingMaskIntoConstraints = false
        contentView.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
}

// Показываем контроллер из любого места
func showBottomSheet(from vc: UIViewController) {
    let sheetVC = BottomSheetViewController()
    
    // Опционально: задаём высоту контента
    sheetVC.preferredContentSize = CGSize(width: 0, height: 400) // влияет на .medium
    
    // Показываем
    vc.present(sheetVC, animated: true)
}
```

### Современные альтернативы UISheetPresentationController в 2026 году

| Альтернатива                                  | Когда лучше UISheetPresentationController           | Минимальная версия | Плюсы                                |
| --------------------------------------------- | --------------------------------------------------- | ------------------ | ------------------------------------ |
| **.sheet** + `.presentationDetents` в SwiftUI | Новый проект или [[SwiftUI]]-экран                  | [[iOS]] 16+        | Декларативный стиль, проще настройка |
| **[[UIViewControllerTransitioningDelegate]]** | Полная кастомизация анимации и размера              | iOS 13+            | Максимальная гибкость                |
| **Custom [[UIPresentationController]]**       | Нужен нестандартный размер/анимация/жесты           | iOS 8+             | Полный контроль                      |
| **FloatingPanel** ([[SPM]]-библиотека)        | Очень кастомный bottom sheet с множеством состояний | iOS 13+            | Красивые анимации, snapping          |

### Лучшие практики UISheetPresentationController в 2026 году

- **Всегда** задавайте хотя бы два detent — `.medium()` и `.large()`  
- **Включайте** `prefersGrabberVisible = true` — это стандартный UX iOS  
- **Отключайте** `prefersScrollingExpandsWhenScrolledToEdge`, если контент не должен автоматически растягиваться  
- **Для кастомной высоты** — используйте `.custom { context in ... }` (iOS 16+)  
- **Для динамического изменения высоты** — меняйте `selectedDetentIdentifier` программно  
- **Для SwiftUI** — используйте `.sheet(detents: [.medium, .large])` — UISheetPresentationController нужен только в UIKit  
- **Для доступности** — задавайте `accessibilityLabel` на grabber и контенте  
- **Документируйте** — пишите комментарий:

```swift
/// UISheetPresentationController с двумя высотами и видимым grabber
if let sheet = sheetPresentationController {
    sheet.detents = [.medium(), .large()]
    sheet.prefersGrabberVisible = true
}
```

**Короткий итог 2026**:
> `UISheetPresentationController` — системный контроллер для **bottom sheet** и **resizable modal** экранов.  
> В 2026 году:  
> - ключевые свойства — `detents`, `selectedDetentIdentifier`, `prefersGrabberVisible`  
> - самый популярный паттерн — `.medium()` + `.large()` + grabber  
> - идеален для фильтров, настроек, деталей, комментариев, bottom sheets  
> - в SwiftUI — заменяется на `.sheet(detents:)`  
> - это **единственный нативный** и **самый рекомендуемый** способ сделать современный лист в UIKit  
