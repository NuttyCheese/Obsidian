**UIPreviewTarget** — это вспомогательный класс в [[UIKit]] (доступен с iOS 13), который используется **исключительно внутри [[UITargetedPreview]]**, чтобы точно указать:

- в каком **контейнере** (view) должен отображаться предпросмотр
- в какой **точке** (center) этого контейнера должен находиться центр предпросмотра
- с каким **размером** (size) показывать предпросмотр

По сути — это «якорь» + «позиция» + «масштаб» для предпросмотра.

### Основные свойства UIPreviewTarget

| Свойство    | Тип           | Описание                                                       | Самый частый сценарий           |
| ----------- | ------------- | -------------------------------------------------------------- | ------------------------------- |
| `container` | [[UIView]]    | Контейнер, относительно которого задаётся центр предпросмотра  | Обычно [[self]] или `superview` |
| `center`    | [[CGPoint]]   | Точка в координатах контейнера, где будет центр предпросмотра  | Место долгого нажатия           |
| `size`      | [[CGSize]]`?` | Желаемый размер предпросмотра (если nil — берётся размер view) | Уменьшение/увеличение           |

### Типичные способы создания UIPreviewTarget

```swift
// 1. Самый частый — предпросмотр в месте касания
let location = gesture.location(in: self)           // или touch.location(in: self)
let target = UIPreviewTarget(container: self, center: location)

// 2. Центрированный предпросмотр
let center = CGPoint(x: bounds.midX, y: bounds.midY)
let target = UIPreviewTarget(container: self, center: center)

// 3. С явным размером (например, уменьшенный вариант)
let target = UIPreviewTarget(
    container: self,
    center: location,
    size: CGSize(width: 200, height: 300)
)
```

### Полный реальный пример 2026 года  
(Предпросмотр при долгом нажатии в таблице / коллекции)

```swift
class MessageCell: UITableViewCell, UIEditMenuInteractionDelegate {
    
    private var menuInteraction: UIEditMenuInteraction?
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        if menuInteraction == nil {
            let interaction = UIEditMenuInteraction(delegate: self)
            addInteraction(interaction)
            menuInteraction = interaction
        }
    }
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        // ... формируем UIMenu
        return UIMenu(children: [ /* действия */ ])
    }
    
    // Самое важное — точный предпросмотр
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        
        // 1. Что показываем в предпросмотре
        let previewView = messageContentView ?? self  // можно показать только часть ячейки
        
        // 2. Параметры оформления
        let params = UIPreviewParameters()
        params.backgroundColor = .clear
        params.shadowRadius = 16
        params.shadowOpacity = 0.25
        params.shadowOffset = CGSize(width: 0, height: 8)
        
        // Можно добавить скругление или маску
        // params.visiblePath = UIBezierPath(roundedRect: previewView.bounds, cornerRadius: 16)
        
        // 3. Точное позиционирование — центр в месте долгого нажатия
        let target = UIPreviewTarget(
            container: self,
            center: location,
            size: CGSize(width: bounds.width * 0.9, height: bounds.height * 0.9) // чуть меньше оригинала
        )
        
        return UITargetedPreview(view: previewView, parameters: params, target: target)
    }
}
```

### Лучшие практики UIPreviewTarget в 2026 году

- **container** — почти всегда `self` (ячейка, view, superview)  
- **center** — передавайте точку касания (`gesture.location(in: self)` или `touch.location(in: self)`)  
- **size** — если не указан → берётся размер `view.bounds`  
- **Не делайте** preview слишком большим — он должен быть чуть меньше или равен оригиналу  
- **Используйте** `UIPreviewParameters` для теней, размытия, трансформации — это сильно улучшает визуал  
- **Для [[SwiftUI]]** — эквивалент `.contextMenu(preferredPreview:)` или `.onLongPressGesture(minimumDuration:0.5, perform: {}, preview: {})`  
- **Документируйте** — пишите комментарий:

```swift
/// Предпросмотр ячейки при долгом нажатии — центр в точке касания
let target = UIPreviewTarget(container: self, center: location)
```

**Короткий итог 2026**:
> `UIPreviewTarget` — это **якорь позиционирования** внутри `UITargetedPreview`.  
> В 2026 году:  
> - ключевые параметры — `container`, `center`, `size`  
> - самый популярный сценарий — предпросмотр в месте долгого нажатия (`location`)  
> - используется почти исключительно внутри `UIEditMenuInteraction` и drag & drop  
> - в SwiftUI — аналог `.contextMenu` с указанием preview  
> - это **маленький, но очень важный** класс для точного и красивого предпросмотра  
