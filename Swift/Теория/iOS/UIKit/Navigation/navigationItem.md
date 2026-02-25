**`navigationItem`** — это свойство класса [[UIViewController]], которое предоставляет доступ к **контенту** и **элементам управления** навигационной панели ([[UINavigationBar]]) текущего контроллера.

Это **самый частый** и **самый важный** способ управлять внешним видом и поведением верхней панели в приложениях с [[UINavigationController]].

### Ключевые характеристики (актуально на 2025–2026 годы)

| Свойство / Метод                             | Тип / Возвращает                              | Что контролирует / зачем нужно                                            | Самый частый сценарий               |
| -------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------------- |
| `title`                                      | [[String]]`?`                                 | Основной заголовок экрана (центр навигационной панели)                    | Название экрана                     |
| `largeTitleDisplayMode`                      | `UINavigationItem.LargeTitleDisplayMode`      | Когда показывать большие заголовки (.automatic, .always, .never, .inline) | `.automatic` — современный стандарт |
| `leftBarButtonItem` / `leftBarButtonItems`   | [[UIBarButtonItem]]`?` / `[UIBarButtonItem]?` | Кнопки слева (обычно "Назад", "Закрыть")                                  | Закрытие модального экрана          |
| `rightBarButtonItem` / `rightBarButtonItems` | `UIBarButtonItem?` / `[UIBarButtonItem]?`     | Кнопки справа (обычно "Добавить", "Редактировать", меню)                  | Действия на экране                  |
| `backBarButtonItem`                          | `UIBarButtonItem?`                            | Кастомизация кнопки "Назад" (текст, иконка)                               | "Назад" → "Главное меню"            |
| `prompt`                                     | `String?`                                     | Дополнительный текст над заголовком                                       | Редко (подсказки)                   |
| `hidesBackButton`                            | [[Bool]]                                      | Скрыть кнопку "Назад" полностью                                           | Экран без возврата                  |
| `menu` (iOS 14+)                             | [[UIMenu]]`?`                                 | Контекстное меню для кнопки (ellipsis.circle)                             | Действия в выпадающем меню          |
| `titleView`                                  | `UIView?`                                     | Кастомный view вместо заголовка (логотип, поиск и т.д.)                   | Кастомный заголовок                 |
| `searchController`                           | [[UISearchController]]`?`                     | Интеграция поиска в навигационную панель                                  | Экраны с поиском                    |
| `scrollEdgeAppearance` (через navigationBar) | `UINavigationBarAppearance?`                  | Внешний вид при скролле до верха                                          | Прозрачный / blur                   |

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Базовый заголовок + кнопки (самый частый)

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Заголовок
        navigationItem.title = "Профиль"
        
        // Кнопка "Настройки" справа
        let settings = UIBarButtonItem(
            image: UIImage(systemName: "gearshape"),
            style: .plain,
            target: self,
            action: #selector(openSettings)
        )
        navigationItem.rightBarButtonItem = settings
        
        // Кнопка "Закрыть" слева (для модального экрана)
        let close = UIBarButtonItem(
            title: "Закрыть",
            style: .done,
            target: self,
            action: #selector(dismissSelf)
        )
        navigationItem.leftBarButtonItem = close
    }
    
    @objc private func openSettings() { /* ... */ }
    @objc private func dismissSelf() { dismiss(animated: true) }
}
```

#### 2. Современный стиль с большими заголовками (Large Titles) + Appearance

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Включаем большие заголовки
    navigationController?.navigationBar.prefersLargeTitles = true
    navigationItem.largeTitleDisplayMode = .automatic  // .always / .never / .inline
    
    // Единый стиль (рекомендуется применять в AppDelegate / SceneDelegate)
    let appearance = UINavigationBarAppearance()
    appearance.configureWithOpaqueBackground()
    appearance.backgroundColor = .systemBackground
    appearance.titleTextAttributes = [.foregroundColor: UIColor.label]
    appearance.largeTitleTextAttributes = [.foregroundColor: UIColor.label]
    
    navigationController?.navigationBar.standardAppearance = appearance
    navigationController?.navigationBar.scrollEdgeAppearance = appearance
}
```

#### 3. Кастомное меню в правой кнопке (UIMenu + UIAction — тренд 2025–2026)

```swift
let edit = UIAction(title: "Редактировать", image: UIImage(systemName: "pencil")) { _ in /* ... */ }
let share = UIAction(title: "Поделиться", image: UIImage(systemName: "square.and.arrow.up")) { _ in /* ... */ }
let delete = UIAction(title: "Удалить", attributes: .destructive) { _ in /* ... */ }

let menu = UIMenu(children: [edit, share, delete])

let button = UIBarButtonItem(title: "Действия", image: UIImage(systemName: "ellipsis.circle"), menu: menu)
navigationItem.rightBarButtonItem = button
```

#### 4. Кастомный заголовок (titleView)

```swift
let logo = UIImageView(image: UIImage(named: "appLogo"))
logo.contentMode = .scaleAspectFit
logo.frame = CGRect(x: 0, y: 0, width: 120, height: 44)

navigationItem.titleView = logo
```

### Лучшие практики UINavigationItem в 2026 году

- **Всегда** используйте `UINavigationBarAppearance` для стиля — это единый и современный способ  
- **Включайте** `prefersLargeTitles = true` + `largeTitleDisplayMode = .automatic` — стандартный вид приложений Apple  
- **Для кнопок справа** — предпочитайте `UIMenu` вместо нескольких отдельных `UIBarButtonItem`  
- **Для кнопки "Назад"** — используйте `backBarButtonItem` для кастомного текста  
- **Для поиска** — интегрируйте `UISearchController` через `navigationItem.searchController`  
- **Для [[SwiftUI]]** — используйте `NavigationStack` / `navigationTitle` — `UINavigationItem` нужен только в смешанных проектах  
- **Документируйте** — пишите комментарий «navigationItem — заголовок 'Профиль' + кнопка 'Настройки' справа с UIMenu»

**Короткий итог 2026**:
> `navigationItem` — это **свойство `UIViewController`**, через которое управляют содержимым `UINavigationBar`.  
> В 2026 году:  
> - ключевые свойства — `title`, `largeTitleDisplayMode`, `left/rightBarButtonItem(s)`, `menu`  
> - стиль — через `UINavigationBarAppearance` (standard / scrollEdge)  
> - современный тренд — `UIMenu` для кнопок справа + large titles  
> Это **центральный** и **самый часто используемый** способ работы с навигационной панелью в [[UIKit]].
