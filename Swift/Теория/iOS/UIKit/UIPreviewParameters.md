**UIPreviewParameters** — это класс в [[UIKit]] (с iOS 13+), который используется **исключительно** внутри [[UITargetedPreview]], чтобы настроить **внешний вид и поведение предпросмотра** (preview), показываемого при:

- долгом нажатии → контекстное меню ([[UIEditMenuInteraction]])
- drag & drop ([[UIDragInteraction]], [[UIDropInteraction]])
- кастомных жестах с предпросмотром

Он отвечает за:

- тень (shadow)
- фон и размытие
- видимую область (visible path / clipping)
- трансформацию (scale, rotation)
- режим отображения (hidden parts, border)

Без `UIPreviewParameters` предпросмотр будет выглядеть «плоско» — просто прямоугольник с содержимым view.

### Основные свойства UIPreviewParameters (самые используемые в 2026)

| Свойство                                          | Тип / Значение по умолчанию | Что настраивает                               | Самый частый сценарий         |
| ------------------------------------------------- | --------------------------- | --------------------------------------------- | ----------------------------- |
| `backgroundColor`                                 | [[UIColor]]`?` ([[nil]])    | Цвет фона за предпросмотром                   | `.clear` — прозрачный фон     |
| `shadowOpacity`                                   | [[Float]] (0.0)             | Прозрачность тени (0.0–1.0)                   | 0.2–0.4 — лёгкая тень         |
| `shadowRadius`                                    | [[CGFloat]] (3.0)           | Радиус размытия тени                          | 8–16 pt — современный эффект  |
| `shadowOffset`                                    | [[CGSize]] (0, 2)           | Смещение тени                                 | CGSize(width: 0, height: 4–8) |
| `visiblePath`                                     | [[UIBezierPath]]`?`         | Видимая область (маска/клиппинг)              | Скруглённые углы, круг        |
| `usesStandardAppearance` (iOS 15+)                | [[Bool]] (true)             | Использовать системный стиль (тень, размытие) | `false` — для кастомного вида |
| `borderWidth` / `borderColor` (через visiblePath) | —                           | Граница вокруг предпросмотра                  | Редко, для выделения          |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Красивый предпросмотр ячейки в таблице при долгом нажатии)

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
        // ... формируем меню
        return UIMenu(children: [ /* действия */ ])
    }
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        
        // 1. Что показываем
        let previewView = contentView  // или messageBubbleView
        
        // 2. Настраиваем параметры — здесь вся магия
        let params = UIPreviewParameters()
        
        // Прозрачный фон
        params.backgroundColor = .clear
        
        // Красивая тень
        params.shadowOpacity = 0.35
        params.shadowRadius = 16
        params.shadowOffset = CGSize(width: 0, height: 8)
        
        // Скруглённые углы через маску
        params.visiblePath = UIBezierPath(
            roundedRect: previewView.bounds,
            cornerRadius: 16
        )
        
        // Опционально: лёгкое масштабирование
        // params.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
        
        // 3. Точное позиционирование
        let target = UIPreviewTarget(
            container: self,
            center: location
        )
        
        return UITargetedPreview(
            view: previewView,
            parameters: params,
            target: target
        )
    }
}
```

### Сравнение: с параметрами vs без

| Ситуация                                  | Без UIPreviewParameters                              | С хорошими UIPreviewParameters                       | Разница |
|-------------------------------------------|-------------------------------------------------------|-------------------------------------------------------|---------|
| Внешний вид                               | Плоский прямоугольник                                 | Тень, скругление, прозрачный фон                     | Значительно красивее |
| Производительность                        | Минимальная                                           | Почти не влияет                                       | Незначительно |
| Пользовательский опыт                     | «Сырой» и дешёвый                                     | Современный, системный, как в Apple Music / Notes     | Очень заметно |
| Поддержка тёмной темы                     | Автоматическая                                        | Полная (системные цвета + shadow)                     | — |

### Лучшие практики UIPreviewParameters в 2026 году

- **backgroundColor = .clear** — почти всегда, чтобы не было белого/чёрного прямоугольника  
- **shadowOpacity 0.2–0.4**, **shadowRadius 10–20**, **shadowOffset height 4–10** — современный «парящий» эффект  
- **visiblePath** — используйте для скруглённых углов или нестандартных форм  
- **size** в `UIPreviewTarget` — делайте чуть меньше оригинала (0.9–0.95) — выглядит элегантнее  
- **Не перегружайте** — тяжёлые view (с видео, сложными анимациями) в предпросмотре тормозят  
- **Для SwiftUI** — аналог `.contextMenu(preferredPreview:)` или `.onLongPressGesture` с `preview` — `UITargetedPreview` нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// Параметры предпросмотра: тень, скругление, прозрачный фон
let params = UIPreviewParameters()
params.backgroundColor = .clear
params.shadowOpacity = 0.3
params.shadowRadius = 16
params.visiblePath = UIBezierPath(roundedRect: bounds, cornerRadius: 16)
```

**Короткий итог 2026**:
> `UIPreviewParameters` — объект стилизации **предпросмотра** внутри `UITargetedPreview`.  
> В 2026 году:  
> - ключевые свойства — `backgroundColor`, `shadowOpacity`, `shadowRadius`, `visiblePath`  
> - самый популярный сценарий — красивый предпросмотр в `UIEditMenuInteraction`  
> - используется для контекстных меню, drag & drop, кастомных long press  
> - в SwiftUI — эквивалент `.contextMenu` с настройкой preview  
> - это **маленький, но критически важный** класс для современного и красивого UX  
