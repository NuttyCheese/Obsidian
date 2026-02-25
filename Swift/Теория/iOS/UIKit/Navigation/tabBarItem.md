**`tabBarItem`** — это свойство класса **[[UIViewController]]**, через которое контроллер представления сообщает **[[UITabBarController]]**, как он должен выглядеть в нижней вкладке (tab bar).

Это **единственный** официальный способ кастомизировать иконку, заголовок, бейдж и поведение вкладки для каждого экрана в таббаре.

### Ключевые характеристики tabBarItem (актуально на 2025–2026 годы)

| Свойство / Метод                           | Тип / Возвращает         | Что контролирует / зачем нужно                    | Самый частый сценарий             |
| ------------------------------------------ | ------------------------ | ------------------------------------------------- | --------------------------------- |
| `title`                                    | [[String]]`?`            | Текст под иконкой вкладки                         | Название вкладки                  |
| `image`                                    | [[UIImage]]`?`           | Иконка в нормальном состоянии                     | Основная иконка                   |
| `selectedImage`                            | `UIImage?`               | Иконка в выделенном состоянии                     | Часто совпадает с image           |
| `badgeValue`                               | `String?`                | Красный бейдж с числом/текстом ([[nil]] = скрыть) | Кол-во непрочитанных сообщений    |
| `badgeColor`                               | [[UIColor]]`?` (iOS 10+) | Цвет бейджа (по умолчанию красный)                | Кастомизация под бренд            |
| `tag`                                      | [[Int]]                  | Произвольный идентификатор вкладки                | Для определения выбранной вкладки |
| `accessibilityLabel` / `accessibilityHint` | `String?`                | Доступность (VoiceOver)                           | Обязательно для доступности       |
| `landscapeImagePhone`                      | `UIImage?`               | Иконка в ландшафтной ориентации на iPhone         | Редко (старые приложения)         |
| `titlePositionAdjustment`                  | `UIOffset`               | Смещение текста под иконкой                       | Точная подгонка дизайна           |

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Базовая настройка вкладки (самый частый)

```swift
class HomeViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Самый простой и рекомендуемый способ
        tabBarItem = UITabBarItem(
            title: "Главная",
            image: UIImage(systemName: "house"),
            tag: 0
        )
        
        // Или отдельно
        tabBarItem.title = "Главная"
        tabBarItem.image = UIImage(systemName: "house")
        tabBarItem.selectedImage = UIImage(systemName: "house.fill")
    }
}
```

#### 2. Современный стиль с SF Symbols + бейджем (2025–2026 стандарт)

```swift
class MessagesViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tabBarItem = UITabBarItem(
            title: "Сообщения",
            image: UIImage(systemName: "message"),
            selectedImage: UIImage(systemName: "message.fill"),
            tag: 1
        )
        
        // Бейдж с количеством непрочитанных
        updateBadge(count: 5)
    }
    
    func updateBadge(count: Int) {
        tabBarItem.badgeValue = count > 0 ? "\(count)" : nil
        
        // Кастомный цвет бейджа (опционально)
        tabBarItem.badgeColor = .systemRed
    }
}
```

#### 3. Кастомная иконка + смещение текста

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let item = UITabBarItem(title: "Профиль", image: UIImage(named: "profile"), tag: 3)
        
        // Смещение текста вниз на 2 pt (иногда нужно для идеального дизайна)
        item.titlePositionAdjustment = UIOffset(horizontal: 0, vertical: 2)
        
        tabBarItem = item
    }
}
```

#### 4. Динамическое обновление бейджа (очень популярный паттерн)

```swift
extension UIViewController {
    func setTabBarBadge(count: Int) {
        tabBarItem.badgeValue = count > 0 ? "\(count)" : nil
        tabBarItem.badgeColor = count > 10 ? .systemOrange : .systemRed
    }
}

// Использование из любого контроллера вкладки
setTabBarBadge(count: unreadMessagesCount)
```

### Лучшие практики tabBarItem в Swift 2026

- **Всегда** используйте **SF Symbols** (`systemName:`) — векторные, адаптивные, поддерживают weight и hierarchy  
- **Для текста** — используйте `.preferredFont(forTextStyle: .caption1)` или системные стили — адаптивно к Dynamic Type  
- **Для бейджа** — обновляйте `badgeValue` динамически, используйте `nil` для скрытия  
- **Для цвета бейджа** — `badgeColor` — кастомизация под бренд (но не злоупотребляйте)  
- **Для accessibility** — задавайте `accessibilityLabel` и `accessibilityHint`  
- **Для SwiftUI** — используйте `TabView` + `tabItem { Label("Главная", systemImage: "house") }` — `tabBarItem` нужен только в смешанных проектах  
- **Документируйте** — пишите комментарий «tabBarItem — вкладка "Главная" с иконкой house и бейджем непрочитанных сообщений»

**Короткий итог 2026**:
> `tabBarItem` — это **свойство UIViewController**, через которое задаётся внешний вид и поведение вкладки в UITabBarController.  
> В 2026 году:  
> - ключевые свойства — `title`, `image`/`selectedImage`, `badgeValue`, `badgeColor`  
> - иконки — SF Symbols (`systemName:`)  
> - бейдж — `badgeValue = "5"` или `nil` для скрытия  
> - это **единственный** официальный способ кастомизировать вкладки в таббаре UIKit  
