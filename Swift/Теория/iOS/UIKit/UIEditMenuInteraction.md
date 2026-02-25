**UIEditMenuInteraction** — это современный механизм в [[UIKit]] (с iOS 13, но значительно улучшен в iOS 16+), который заменяет ==устаревший== [[UIMenuController]] для реализации **контекстных меню** (edit menu) при долгом нажатии на текст, изображения, ячейки таблиц и любые другие [[UIView]].

Он позволяет:

- показывать стандартное меню редактирования (Copy, Cut, Paste, Select All и т.д.)
- добавлять **кастомные пункты** и **подменю**
- управлять **предпросмотром** (preview) при долгом нажатии
- динамически генерировать меню в зависимости от контекста
- поддерживать **жесты**, **accessibility** и **новые возможности** iOS (например, меню в macOS Catalyst)

### Почему UIEditMenuInteraction — это стандарт 2026 года

| Старый подход (UIMenuController)      | Новый подход (UIEditMenuInteraction)        | Преимущества нового подхода       |
| ------------------------------------- | ------------------------------------------- | --------------------------------- |
| Устарел с iOS 13                      | Активно развивается Apple                   | Поддержка новых фич iOS           |
| Ограниченные кастомные пункты         | Полноценные UIMenu + [[UIAction]] + подменю | Гибкость, динамика                |
| Нет предпросмотра                     | Поддержка [[UITargetedPreview]]             | Красивый UX                       |
| Сложно управлять состоянием           | Легко проверять canPerformAction / validate | Динамическое включение/выключение |
| Плохо работает с современными жестами | Полная интеграция с жестами и accessibility | Лучшая доступность                |

### Как добавить UIEditMenuInteraction (самый популярный паттерн 2026)

```swift
import UIKit

class CustomTextView: UITextView {
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        // Добавляем интеракцию один раз
        if interactions.first(where: { $0 is UIEditMenuInteraction }) == nil {
            let interaction = UIEditMenuInteraction(delegate: self)
            addInteraction(interaction)
        }
    }
}

// MARK: - UIEditMenuInteractionDelegate
extension CustomTextView: UIEditMenuInteractionDelegate {
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        
        // Кастомные действия
        let copyWithDate = UIAction(title: "Копировать с датой") { _ in
            let date = DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .short)
            UIPasteboard.general.string = "\(self.text ?? "") — \(date)"
        }
        
        let uppercase = UIAction(title: "В ВЕРХНИЙ РЕГИСТР") { _ in
            self.text = self.text?.uppercased()
        }
        
        let share = UIAction(title: "Поделиться", image: UIImage(systemName: "square.and.arrow.up")) { _ in
            let activityVC = UIActivityViewController(activityItems: [self.text ?? ""], applicationActivities: nil)
            self.window?.rootViewController?.present(activityVC, animated: true)
        }
        
        // Можно сделать подменю
        let formatMenu = UIMenu(title: "Форматирование", children: [
            UIAction(title: "Жирный") { _ in /* применить атрибут */ },
            UIAction(title: "Курсив") { _ in /* применить атрибут */ }
        ])
        
        return UIMenu(title: "", options: .displayInline, children: [
            copyWithDate,
            uppercase,
            formatMenu,
            share
        ])
    }
    
    // Опционально: предпросмотр при долгом нажатии
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        
        // Можно вернуть кастомный предпросмотр выделенного текста
        let previewParams = UIPreviewParameters()
        previewParams.backgroundColor = .clear
        
        return UITargetedPreview(view: self, parameters: previewParams)
    }
}
```

### Ключевые моменты 2026 года

- **Добавляйте** `UIEditMenuInteraction` один раз через `addInteraction(_:)` (лучше в `didMoveToWindow` или [[viewDidLoad]])
- **Не используйте** ==старый== `UIMenuController` и [[UIMenuItem]] — они устарели и не поддерживают новые возможности
- **Для динамики** — меню генерируется заново каждый раз при долгом нажатии → всегда актуально
- **Для кастомного вида** — используйте `previewForMenuAt` для предпросмотра
- **Для SwiftUI** — используйте `.contextMenu { ... }` или `.menu` — `UIEditMenuInteraction` нужен только в UIKit
- **Для доступности** — добавляйте `accessibilityLabel` и `accessibilityHint` в `UIAction`
- **Документируйте** — пишите комментарий:

```swift
/// UIEditMenuInteraction для кастомного контекстного меню при выделении текста
let interaction = UIEditMenuInteraction(delegate: self)
addInteraction(interaction)
```

**Короткий итог 2026**:
> `UIEditMenuInteraction` — современный механизм для **контекстных меню** при долгом нажатии (копировать, вырезать, кастомные действия).  
> В 2026 году:  
> - полностью заменил устаревший `UIMenuController`  
> - использует [[UIMenu]] + [[UIAction]] для гибких и динамических меню  
> - ключевой метод — `editMenuInteraction(_:menuFor:at:)`  
> - поддерживает предпросмотр, подменю, состояния и новые жесты  
> - это **единственный рекомендуемый** способ кастомизировать контекстное меню в UIKit  
