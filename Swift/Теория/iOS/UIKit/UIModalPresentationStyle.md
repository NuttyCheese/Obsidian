**UIModalPresentationStyle** — это перечисление ([[enum]]) в [[UIKit]], которое определяет, **как именно** будет отображаться модальный контроллер при вызове `present(_:animated:completion:)`.

Оно задаёт **стиль презентации** (полноэкранный, лист снизу, карточка, popover, текущий контекст и т.д.), а также влияет на:

- поведение свайпа для закрытия  
- адаптацию под iPhone/iPad  
- взаимодействие с предыдущим экраном (затемнение, прозрачность)  
- возможность изменения размера и позиции

### Все значения UIModalPresentationStyle (актуально на 2026 год)

| Значение                              | iOS с которой доступно | Как выглядит на iPhone / iPad                               | Когда использовать в 2026 | Самый частый сценарий |
|---------------------------------------|-------------------------|-------------------------------------------------------------|----------------------------|-----------------------|
| `.automatic`                          | iOS 13+                 | Система сама выбирает (обычно `.pageSheet` на iPad, `.fullScreen` на iPhone) | Почти всегда по умолчанию  | Универсальный выбор |
| `.fullScreen`                         | iOS 3+                  | Полноэкранный модал, закрывается только кнопкой или жестом  | Когда нужен захват всего экрана | Формы, авторизация, полноэкранные модалки |
| `.pageSheet`                          | iOS 15+ (но работает раньше) | Лист снизу, можно тянуть вниз для закрытия, меняет высоту   | Самый популярный в 2026    | Bottom sheet, настройки, фильтры, детали |
| `.formSheet`                          | iOS 3+                  | Центрированная карточка (особенно на iPad)                  | Редко на iPhone            | Классические формы на iPad |
| `.currentContext`                     | iOS 8+                  | Поверх текущего контроллера (не закрывает предыдущий)       | Вложенные модалки          | Модал внутри navigation stack |
| `.overCurrentContext`                 | iOS 8+                  | Поверх текущего, с затемнением, но не закрывает стек        | Полупрозрачные оверлеи     | Alert-подобные окна, быстрые действия |
| `.overFullScreen`                     | iOS 8+                  | Полноэкранный оверлей, не влияет на предыдущий контроллер   | Полноэкранные модалки без влияния на стек | Splash, onboarding, loading |
| `.popover`                            | iOS 8+                  | Popover (стрелка от источника) — только на iPad             | Редко (заменён .pageSheet) | Старые popover-меню |
| `.custom`                             | iOS 7+                  | Полностью кастомный стиль (требует `transitioningDelegate`) | Когда нужен уникальный вид | Кастомные bottom sheet, card, zoom |
| `.none`                               | iOS 7+                  | Без анимации и без модальности (редко)                      | Очень редко                | Специфические хаки |

### Самый популярный и рекомендуемый стиль в 2026 году

```swift
.sheet / .pageSheet
```

```swift
let vc = DetailViewController()
vc.modalPresentationStyle = .pageSheet

if let sheet = vc.sheetPresentationController {
    sheet.detents = [.medium(), .large()]
    sheet.prefersGrabberVisible = true
    sheet.prefersScrollingExpandsWhenScrolledToEdge = false
    sheet.largestUndimmedDetentIdentifier = .large
    sheet.prefersEdgeAttachedInCompactHeight = true
    sheet.preferredCornerRadius = 24
}

present(vc, animated: true)
```

### Полный современный пример (iOS 15+ / 2026 стиль)

```swift
class FilterViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        if let sheet = sheetPresentationController {
            sheet.detents = [
                .custom { context in UISheetPresentationController.Detent.Value(300) }, // фиксированная высота
                .medium(),
                .large()
            ]
            sheet.selectedDetentIdentifier = .medium
            sheet.prefersGrabberVisible = true
            sheet.prefersDimmedScrimVisible = true
            sheet.prefersScrollingExpandsWhenScrolledToEdge = false
            sheet.widthFollowsPreferredContentSizeWhenEdgeAttached = true
            sheet.preferredCornerRadius = 28
        }
        
        // Контент контроллера
        // ...
    }
}

// Показ из любого места
func showFilter(from vc: UIViewController) {
    let filterVC = FilterViewController()
    filterVC.modalPresentationStyle = .pageSheet
    vc.present(filterVC, animated: true)
}
```

### Лучшие практики [[UISheetPresentationController]] в 2026 году

- **Предпочитайте** `.pageSheet` — это современный стандарт Apple для модальных листов  
- **Всегда** задавайте хотя бы `.medium()` и `.large()` в `detents`  
- **Включайте** `prefersGrabberVisible = true` — это улучшает UX  
- **Контролируйте затемнение** через `largestUndimmedDetentIdentifier`  
- **Для динамической высоты** — используйте `.custom { context in ... }`  
- **Для [[SwiftUI]]** — используйте `.sheet(detents: [.medium, .large])` — `UISheetPresentationController` нужен только в [[UIKit]]  
- **Для доступности** — задавайте `accessibilityLabel` на grabber и контенте  
- **Документируйте** — пишите комментарий:

```swift
/// Модальный лист с изменяемой высотой (medium / large)
vc.modalPresentationStyle = .pageSheet
if let sheet = vc.sheetPresentationController {
    sheet.detents = [.medium(), .large()]
    sheet.prefersGrabberVisible = true
}
```

**Короткий итог 2026**:
> `UIModalPresentationStyle` — определяет **стиль отображения** модального контроллера при `present`.  
> В 2026 году:  
> - самый популярный — `.pageSheet` (bottom sheet, resizable)  
> - ключевые стили — `.automatic`, `.fullScreen`, `.pageSheet`, `.custom`  
> - для полной кастомизации — `.custom` + `UISheetPresentationController` + `transitioningDelegate`  
> - в SwiftUI — эквивалент `.sheet(detents:)`  
> - это **основа** современных модальных окон в iOS-приложениях  
