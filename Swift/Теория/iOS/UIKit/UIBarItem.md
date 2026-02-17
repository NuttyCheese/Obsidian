**UIBarItem** — это **абстрактный базовый класс** в [[UIKit]], который описывает любой элемент, который может отображаться **на панелях** (bars):

- **UITabBar** (нижняя панель вкладок)
- **UIToolbar** (панель инструментов, обычно снизу)
- **UINavigationBar** (верхняя навигационная панель)

Сам по себе `UIBarItem` **не создаётся** и **не используется** напрямую — это родительский класс для двух конкретных типов:

| Класс                   | Где используется                                  | Основное назначение в 2026 году                                              |
| ----------------------- | ------------------------------------------------- | ---------------------------------------------------------------------------- |
| **[[UIBarButtonItem]]** | [[UINavigationBar]] (слева/справа), [[UIToolbar]] | Кнопки действий: «Сохранить», «Редактировать», «Добавить», иконки SF Symbols |
| **[[UITabBarItem]]**    | [[UITabBar]] (вкладки внизу экрана)               | Вкладки: «Главная», «Профиль», «Поиск» и т.д.                                |

### Ключевые свойства UIBarItem (общие для обоих подклассов)

| Свойство              | Тип          | Что делает / Когда использовать в 2026           | Пример                              |
| --------------------- | ------------ | ------------------------------------------------ | ----------------------------------- |
| `title`               | [[String]]?  | Текст элемента                                   | `tabBarItem.title = "Главная"`      |
| `image`               | [[UIImage]]? | Иконка в обычном состоянии                       | `UIImage(systemName: "house")`      |
| `selectedImage`       | `UIImage?`   | Иконка в выбранном состоянии (для вкладок)       | `UIImage(systemName: "house.fill")` |
| `badgeValue`          | `String?`    | Красный бейдж с числом/текстом                   | `tabBarItem.badgeValue = "5"`       |
| `badgeColor`          | [[UIColor]]? | Цвет бейджа                                      | `.systemRed`                        |
| `tag`                 | [[Int]]      | Идентификатор для отличия элементов              | `barButton.tag = 42`                |
| `isEnabled`           | [[Bool]]     | Доступность (серая/активная)                     | `isEnabled = false` при загрузке    |
| `landscapeImagePhone` | `UIImage?`   | Иконка для ландшафтного режима на iPhone (редко) | —                                   |
| `accessibilityLabel`  | `String?`    | Голосовой доступ (VoiceOver)                     | Обязательно для доступности         |

### Самые частые сценарии использования в 2026 году

#### 1. UIBarButtonItem в Navigation Bar (самый популярный случай)

```swift
class DetailViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Кнопка "Сохранить" справа
        let save = UIBarButtonItem(
            title: "Сохранить",
            style: .done,
            target: self,
            action: #selector(saveTapped)
        )
        navigationItem.rightBarButtonItem = save
        
        // Кнопка "Назад" слева с иконкой
        let back = UIBarButtonItem(
            image: UIImage(systemName: "chevron.left"),
            style: .plain,
            target: self,
            action: #selector(backTapped)
        )
        navigationItem.leftBarButtonItem = back
    }
    
    @objc private func saveTapped() { /* сохранение */ }
    @objc private func backTapped() { navigationController?.popViewController(animated: true) }
}
```

#### 2. Несколько кнопок справа (группа действий)

```swift
let share = UIBarButtonItem(barButtonSystemItem: .action, target: self, action: #selector(shareTapped))
let edit = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))

navigationItem.rightBarButtonItems = [share, edit]
```

#### 3. UITabBarItem (вкладки внизу)

```swift
let homeTab = UITabBarItem(
    title: "Главная",
    image: UIImage(systemName: "house"),
    selectedImage: UIImage(systemName: "house.fill")
)
homeTab.badgeValue = "3"  // красный бейдж с числом 3

let profileTab = UITabBarItem(
    title: "Профиль",
    image: UIImage(systemName: "person"),
    selectedImage: UIImage(systemName: "person.fill")
)

let tabBarController = UITabBarController()
tabBarController.viewControllers = [
    UINavigationController(rootViewController: HomeVC()).then { $0.tabBarItem = homeTab },
    UINavigationController(rootViewController: ProfileVC()).then { $0.tabBarItem = profileTab }
]
```

#### 4. Кастомная вью в UIBarButtonItem (редко, но мощно)

```swift
let badgeLabel = UILabel()
badgeLabel.text = "New"
badgeLabel.textColor = .white
badgeLabel.backgroundColor = .systemRed
badgeLabel.font = .systemFont(ofSize: 12, weight: .bold)
badgeLabel.textAlignment = .center
badgeLabel.layer.cornerRadius = 10
badgeLabel.clipsToBounds = true
badgeLabel.frame = CGRect(x: 0, y: 0, width: 40, height: 20)

let customButton = UIBarButtonItem(customView: badgeLabel)
navigationItem.rightBarButtonItem = customButton
```

### Лучшие практики UIBarItem в Swift 2026

- **Используй UIAction вместо target-action** (iOS 14+)

```swift
let saveAction = UIAction { _ in saveDocument() }
let saveButton = UIBarButtonItem(title: "Сохранить", primaryAction: saveAction)
navigationItem.rightBarButtonItem = saveButton
```

- **Системные иконки** — всегда `barButtonSystemItem` (`.edit`, `.add`, `.done`, `.trash`, `.action`, `.refresh` и т.д.) — они автоматически адаптируются под тему, доступность и размер  
- **SF Symbols** — используй `UIImage(systemName:weight:scale:)` для кастомных иконок  
- **Максимум 2–3 кнопки справа** — больше выглядит перегруженно  
- **iPad / Mac Catalyst** — обязательно указывай `popoverPresentationController` при показе action sheet из кнопки  
- **[[@MainActor]]** — все действия кнопок — на главном акторе  
- **[[Swift]] 6 strict concurrency** — [[UIBarButtonItem]] полностью безопасен  
- **Документируйте** — пиши комментарий «UIBarButtonItem — кнопка "Сохранить" в навигационной панели»

**Короткий девиз 2026**:
> UIBarItem — это **абстрактный элемент** для кнопок и вкладок на панелях (навигация, тулбар, таб-бар).  
> В 2026 году используй **[[UIAction]]**, системные иконки и максимум 2–3 кнопки справа.  
> Это **единственный правильный** способ добавить действия в верхнюю/нижнюю панель в UIKit.
