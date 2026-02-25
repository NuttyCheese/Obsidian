**UITargetedPreview** — это класс в [[UIKit]] (доступен с **iOS 13**, 2019), который используется для создания **предпросмотра** (preview) при взаимодействии с элементами интерфейса, особенно в следующих сценариях:

- контекстное меню (UIEditMenuInteraction)
- drag & drop (`UIDragInteraction`, `UIDropInteraction`)
- peek & pop / force touch (устаревший 3D Touch, но API остался)
- кастомные preview при долгом нажатии

Он позволяет указать:

- какой **view** показать в предпросмотре
- **размер** и **положение** предпросмотра
- **эффекты** (тень, размытие, трансформацию)
- **параметры** анимации и стиля

Это **ключевой** компонент для создания современных, красивых и интуитивных взаимодействий в iOS-приложениях 2026 года.

### Основные свойства UITargetedPreview

| Свойство                          | Тип / Значение по умолчанию                          | Что задаёт / зачем нужно                                      | Самый частый сценарий |
|-----------------------------------|-------------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `view`                            | `UIView`                                              | Представление, которое будет показано в предпросмотре         | Основной элемент предпросмотра |
| `parameters`                      | `UIPreviewParameters`                                 | Стилизация: тень, размытие, клиппинг, трансформация           | Настройка внешнего вида |
| `target`                          | `UIPreviewTarget`                                     | Положение и якорь предпросмотра на экране                     | Точное позиционирование |
| `size` (в target)                 | `CGSize`                                              | Размер предпросмотра                                          | Уменьшение/увеличение |
| `center` (в target)               | `CGPoint`                                             | Центр предпросмотра относительно контейнера                   | Выравнивание по точке касания |

### Самый популярный паттерн 2026 года  
(Предпросмотр при долгом нажатии в UIEditMenuInteraction)

```swift
import UIKit

class MessageCell: UITableViewCell {
    
    private var menuInteraction: UIEditMenuInteraction?
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        if menuInteraction == nil {
            let interaction = UIEditMenuInteraction(delegate: self)
            addInteraction(interaction)
            menuInteraction = interaction
        }
    }
}

// MARK: - UIEditMenuInteractionDelegate
extension MessageCell: UIEditMenuInteractionDelegate {
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        
        let reply = UIAction(title: "Ответить") { _ in
            // ...
        }
        
        let copy = UIAction(title: "Копировать") { _ in
            UIPasteboard.general.string = self.messageText
        }
        
        return UIMenu(title: "", children: [reply, copy])
    }
    
    // Самый важный метод — предпросмотр при долгом нажатии
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        
        // 1. Берём view, который хотим показать (например, весь cell или label с текстом)
        let previewView = messageLabel ?? self
        
        // 2. Настраиваем параметры предпросмотра
        let parameters = UIPreviewParameters()
        parameters.backgroundColor = .clear
        parameters.shadowOpacity = 0.3
        parameters.shadowRadius = 12
        parameters.shadowOffset = CGSize(width: 0, height: 4)
        
        // Можно добавить маску или клиппинг
        // parameters.visiblePath = UIBezierPath(roundedRect: previewView.bounds, cornerRadius: 12)
        
        // 3. Указываем точку привязки (target) — обычно место долгого нажатия
        let target = UIPreviewTarget(container: self, center: location)
        
        // 4. Создаём и возвращаем предпросмотр
        return UITargetedPreview(view: previewView, parameters: parameters, target: target)
    }
}
```

### Другие популярные сценарии использования UITargetedPreview

1. **Drag & Drop** (UIDragInteraction)

```swift
func dragInteraction(_ interaction: UIDragInteraction, previewForCancelling itemProvider: UIDragPreview) -> UITargetedPreview? {
    let target = UIPreviewTarget(container: view, center: CGPoint(x: view.bounds.midX, y: view.bounds.midY))
    let params = UIPreviewParameters()
    params.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
    return UITargetedPreview(view: draggedView, parameters: params, target: target)
}
```

2. **Кастомный контекстный жест** (UILongPressGestureRecognizer)

```swift
@objc func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
    guard gesture.state == .began else { return }
    
    let location = gesture.location(in: view)
    let preview = UITargetedPreview(view: someSubview, parameters: UIPreviewParameters())
    
    // Показываем предпросмотр вручную (редко)
    // обычно используется в UIEditMenuInteraction
}
```

### Лучшие практики UITargetedPreview в 2026 году

- **Всегда** указывайте `view` — это то, что будет показано в предпросмотре  
- **Используйте** `UIPreviewParameters` для теней, размытия, трансформации, маски  
- **Для точного позиционирования** — применяйте `UIPreviewTarget` с `center` в точке касания  
- **Для SwiftUI** — используйте `.contextMenu(preferredPreview:)` или `.onLongPressGesture` с `preview` — `UITargetedPreview` нужен только в UIKit  
- **Для производительности** — не используйте сложные view с анимациями в предпросмотре  
- **Для доступности** — предпросмотр должен поддерживать VoiceOver (accessibility)  
- **Документируйте** — пишите комментарий:

```swift
/// Предпросмотр при долгом нажатии на ячейку сообщения
func editMenuInteraction(_ interaction: UIEditMenuInteraction, previewForMenuAt location: CGPoint) -> UITargetedPreview? {
    // ...
}
```

**Короткий итог 2026**:
> `UITargetedPreview` — объект для создания **красивого предпросмотра** при долгом нажатии, drag & drop и других взаимодействиях.  
> В 2026 году:  
> - ключевые компоненты — `view`, `parameters`, `target`  
> - самый популярный сценарий — предпросмотр в `UIEditMenuInteraction`  
> - используется для контекстных меню, drag preview, кастомных жестов  
> - в SwiftUI — эквивалент `.contextMenu` с `preview`  
> - это **must-have** для современного, отзывчивого и красивого UX в UIKit  

Удачи с интуитивными и визуально приятными предпросмотрами в твоём приложении! 👀