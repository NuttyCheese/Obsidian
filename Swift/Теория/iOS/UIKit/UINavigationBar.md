**UINavigationBar** — это верхняя панель навигации в [[iOS]]-приложениях.  
Она является визуальной частью **[[UINavigationController]]** и отвечает за:

- отображение **заголовка** текущего экрана (title / large title)
- размещение **кнопок навигации** слева и справа (back button, close, edit, add, custom actions)
- показ **prompt** (дополнительный текст над заголовком)
- визуальную обратную связь при скролле (появление/исчезновение, blur, large titles → standard)

В 2025–2026 годах UINavigationBar остаётся **центральным элементом** навигации в UIKit-приложениях.  
Она полностью поддерживает **тёмную тему**, **Dynamic Type**, **Large Titles**, **scroll edge appearance** и **customization через [[UINavigationBarAppearance]]**.

### Основные элементы UINavigationBar

| Элемент                              | Управляется через                           | Что отображает / зачем нужен                              | Самый частый сценарий |
|--------------------------------------|---------------------------------------------|------------------------------------------------------------|-----------------------|
| `title`                              | `navigationItem.title`                      | Основной заголовок экрана                                  | Название экрана |
| `largeTitleDisplayMode`              | `navigationItem.largeTitleDisplayMode`      | Когда показывать большой заголовок (.automatic, .always, .never) | `.automatic` — стандарт |
| `leftBarButtonItem(s)`               | `navigationItem.leftBarButtonItem` / `Items` | Кнопка(и) слева (обычно "Назад", "Закрыть")                | Закрытие модального экрана |
| `rightBarButtonItem(s)`              | `navigationItem.rightBarButtonItem` / `Items` | Кнопка(и) справа (обычно "Добавить", "Редактировать")      | Действия на экране |
| `prompt`                             | `navigationItem.prompt`                     | Дополнительный текст над заголовком                        | Редко (подсказки) |
| `backBarButtonItem`                  | `navigationItem.backBarButtonItem`          | Кастомизация текста кнопки "Назад"                         | "Назад" → "Главное меню" |
| `scrollEdgeAppearance`               | `navigationBar.scrollEdgeAppearance`        | Внешний вид при скролле до самого верха                    | Прозрачный / blur |
| `standardAppearance`                 | `navigationBar.standardAppearance`          | Обычный вид (когда контент прокручен)                      | Основной стиль |
| `compactAppearance`                  | `navigationBar.compactAppearance`           | Вид в компактном состоянии (landscape iPhone)              | Редко кастомизируют |

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Базовая настройка заголовка и кнопок

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Заголовок
        navigationItem.title = "Профиль"
        
        // Кнопка "Настройки" справа
        let settingsButton = UIBarButtonItem(
            image: UIImage(systemName: "gearshape"),
            style: .plain,
            target: self,
            action: #selector(openSettings)
        )
        navigationItem.rightBarButtonItem = settingsButton
        
        // Кнопка "Закрыть" слева (для модального экрана)
        let closeButton = UIBarButtonItem(
            title: "Закрыть",
            style: .done,
            target: self,
            action: #selector(dismissSelf)
        )
        navigationItem.leftBarButtonItem = closeButton
    }
    
    @objc private func openSettings() { /* ... */ }
    @objc private func dismissSelf() { dismiss(animated: true) }
}
```

#### 2. Современный стиль с Large Titles и Appearance (самый рекомендуемый)

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Включаем большие заголовки
    navigationController?.navigationBar.prefersLargeTitles = true
    navigationItem.largeTitleDisplayMode = .automatic  // .always / .never / .inline
    
    // Единый стиль для всей навигации
    let appearance = UINavigationBarAppearance()
    appearance.configureWithOpaqueBackground()
    appearance.backgroundColor = .systemBackground
    appearance.titleTextAttributes = [
        .foregroundColor: UIColor.label,
        .font: UIFont.preferredFont(forTextStyle: .headline)
    ]
    appearance.largeTitleTextAttributes = [
        .foregroundColor: UIColor.label,
        .font: UIFont.preferredFont(forTextStyle: .largeTitle)
    ]
    
    // Применяем ко всем состояниям
    navigationController?.navigationBar.standardAppearance = appearance
    navigationController?.navigationBar.scrollEdgeAppearance = appearance
    navigationController?.navigationBar.compactAppearance = appearance
    
    // Прозрачный бар при скролле (популярный стиль 2025–2026)
    let transparent = UINavigationBarAppearance()
    transparent.configureWithTransparentBackground()
    transparent.backgroundEffect = UIBlurEffect(style: .systemMaterial)
    navigationController?.navigationBar.scrollEdgeAppearance = transparent
}
```

#### 3. Кастомное меню в правой кнопке ([[UIMenu]] + [[UIAction]])

```swift
let menuActions = [
    UIAction(title: "Редактировать", image: UIImage(systemName: "pencil")) { _ in /* ... */ },
    UIAction(title: "Поделиться", image: UIImage(systemName: "square.and.arrow.up")) { _ in /* ... */ },
    UIAction(title: "Удалить", image: UIImage(systemName: "trash"), attributes: .destructive) { _ in /* ... */ }
]

let menu = UIMenu(children: menuActions)

let menuButton = UIBarButtonItem(title: "Действия", image: UIImage(systemName: "ellipsis.circle"), menu: menu)
navigationItem.rightBarButtonItem = menuButton
```

#### 4. Кастомизация кнопки "Назад"

```swift
// Изменяем текст кнопки "Назад" на текущем экране
navigationItem.backBarButtonItem = UIBarButtonItem(title: "Назад к списку", style: .plain, target: nil, action: nil)

// Или полностью кастомная кнопка "Назад"
let customBack = UIBarButtonItem(image: UIImage(systemName: "chevron.left"), style: .plain, target: self, action: #selector(customBackAction))
navigationItem.leftBarButtonItem = customBack
```

### Лучшие практики UINavigationBar в Swift 2026

- **Всегда** используйте `UINavigationBarAppearance` для единообразного стиля (стандарт 2025–2026)  
- **Включайте** `prefersLargeTitles = true` + `largeTitleDisplayMode = .automatic` — это современный вид  
- **Для scroll edge** — делайте прозрачный blur (`scrollEdgeAppearance`) — выглядит премиально  
- **Для кнопок** — предпочитайте `UIMenu` + [[UIBarButtonItem]]`(menu:)` вместо множества отдельных кнопок  
- **Для иконок** — используйте SF Symbols с `.medium` / `.semibold` weight  
- **Для [[SwiftUI]]** — используйте `NavigationStack` / `NavigationBarTitleDisplayMode` — UINavigationBar нужен только в смешанных проектах  
- **Документируйте** — пишите комментарий «UINavigationBarAppearance — единый стиль с blur при скролле и large titles»

**Короткий итог 2026**:
> UINavigationBar — это **верхняя навигационная панель** в UIKit-приложениях.  
> В 2026 году:  
> - управляется через `navigationItem` (title, left/rightBarButtonItems)  
> - кастомизация — через `UINavigationBarAppearance` (standard, scrollEdge, compact)  
> - поддерживает large titles, меню (UIMenu), кнопки, prompt  
> - это **центральный** элемент навигации в любом [[UIKit]]-приложении  
