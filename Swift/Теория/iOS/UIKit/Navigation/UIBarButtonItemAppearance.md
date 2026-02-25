**UIBarButtonItemAppearance** — это объект конфигурации внешнего вида **[[UIBarButtonItem]]** (кнопок в navigation bar, toolbar, tab bar и т.д.), введённый в [[iOS]] 13 (2019). Он позволяет централизованно и декларативно задавать стиль кнопок в разных состояниях и контекстах.

С помощью `UIBarButtonItemAppearance` вы можете управлять:

- цветом текста и иконок  
- шрифтом  
- состоянием (normal, highlighted, disabled, selected)  
- стилем кнопки (plain, done и т.д.)  
- отступами и позиционированием  

Это **современный и рекомендуемый** способ кастомизации кнопок в навигации и тулбарах в 2026 году (заменил старые `tintColor`, `setTitleTextAttributes` и т.д.).

### Основные свойства UIBarButtonItemAppearance

| Свойство                                                       | Тип / Значение по умолчанию                 | Что настраивает                                         | Самый частый сценарий      |
| -------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------- | -------------------------- |
| `normal` / `highlighted` / `disabled` / `focused` / `selected` | `UIBarButtonItemAppearance.StyleAppearance` | Состояния кнопки (обычное, нажатое, отключённое и т.д.) | Разные цвета для состояний |
| `plainAppearance` / `doneAppearance`                           | `UIBarButtonItemAppearance.StyleAppearance` | Специфические стили для `.plain` и `.done` кнопок       | Выделение "Done"           |
| `titleTextAttributes`                                          | `[NSAttributedString.Key: Any]?`            | Атрибуты текста кнопки (шрифт, цвет, кернинг)           | Кастомный шрифт            |
| `backgroundImage`                                              | [[UIImage]]`?`                              | Фоновое изображение кнопки                              | Градиент, кастомный фон    |
| `backgroundVerticalPositionAdjustment`                         | [[CGFloat]]                                 | Смещение по вертикали                                   | Точная подгонка            |
| `tintColor` (через appearance)                                 | [[UIColor]]`?`                              | Глобальный tint для всех кнопок                         | Брендовый цвет             |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Глобальная + локальная кастомизация кнопок)

```swift
import UIKit

class AppAppearance {
    
    static func setupGlobalBarButtonAppearance() {
        
        // Глобальный стиль для всех UIBarButtonItem
        let appearance = UIBarButtonItemAppearance()
        
        // Обычное состояние
        appearance.normal.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .medium),
            .foregroundColor: UIColor.systemBlue
        ]
        
        // Нажатое состояние
        appearance.highlighted.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .medium),
            .foregroundColor: UIColor.systemBlue.withAlphaComponent(0.6)
        ]
        
        // Отключённое состояние
        appearance.disabled.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .medium),
            .foregroundColor: UIColor.systemGray
        ]
        
        // Специально для кнопки "Done"
        let doneAppearance = UIBarButtonItemAppearance(style: .done)
        doneAppearance.normal.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .bold),
            .foregroundColor: UIColor.systemBlue
        ]
        
        // Применяем глобально
        UIBarButtonItem.appearance().standardAppearance = appearance
        UIBarButtonItem.appearance().compactAppearance = appearance  // для landscape iPhone
        UIBarButtonItem.appearance().scrollEdgeAppearance = appearance
        
        // Специально для "Done"
        UIBarButtonItem.appearance(whenContainedInInstancesOf: [UINavigationBar.self]).doneButtonAppearance = doneAppearance
    }
}

// Вызов один раз при запуске (в AppDelegate / SceneDelegate)
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    AppAppearance.setupGlobalBarButtonAppearance()
    return true
}
```

### Локальная кастомизация для конкретного экрана

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Локальный стиль только для этого экрана
        let appearance = UIBarButtonItemAppearance()
        appearance.normal.titleTextAttributes = [
            .foregroundColor: UIColor.white,
            .font: UIFont.systemFont(ofSize: 17, weight: .semibold)
        ]
        
        // Применяем к кнопкам на этом экране
        navigationItem.rightBarButtonItem?.standardAppearance = appearance
        navigationItem.leftBarButtonItem?.standardAppearance = appearance
        
        // Или через navigation bar appearance
        let navAppearance = UINavigationBarAppearance()
        navAppearance.buttonAppearance = appearance
        navigationItem.standardAppearance = navAppearance
    }
}
```

### Лучшие практики UIBarButtonItemAppearance в 2026 году

- **Глобально** настраивайте через `UIBarButtonItem.appearance()` в `didFinishLaunchingWithOptions` или `scene(_:willConnectTo:)`  
- **Обязательно** задавайте стили для `normal`, `highlighted`, `disabled` — иначе кнопки будут выглядеть неконсистентно  
- **Для "Done" кнопок** — используйте `doneButtonAppearance` — они часто выделяются  
- **Для тёмной темы** — используйте системные цвета (`.label`, `.systemBlue`) — они автоматически адаптируются  
- **Для large title** — синхронизируйте с `UINavigationBarAppearance.largeTitleTextAttributes`  
- **Для SwiftUI** — используйте `.toolbar` + `.tint` / `.foregroundStyle` — `UIBarButtonItemAppearance` нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// Глобальная настройка внешнего вида всех UIBarButtonItem
static func setupGlobalBarButtonAppearance() {
    let appearance = UIBarButtonItemAppearance()
    // ...
}
```

**Короткий итог 2026**:
> `UIBarButtonItemAppearance` — объект для **конфигурации внешнего вида** кнопок в navigation bar, toolbar и других местах.  
> В 2026 году:  
> - ключевые свойства — `normal`, `highlighted`, `disabled`, `titleTextAttributes`, `doneButtonAppearance`  
> - глобально настраивается через `UIBarButtonItem.appearance()`  
> - локально — через `navigationItem` или `barButtonItem.standardAppearance`  
> - это **единственный современный** и **рекомендуемый** способ кастомизировать кнопки в UIKit  
