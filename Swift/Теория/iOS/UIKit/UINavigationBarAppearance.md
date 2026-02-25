**UINavigationBarAppearance** — это объект конфигурации внешнего вида **UINavigationBar**, введённый в iOS 13 (2019), который заменил старые свойства `barTintColor`, `titleTextAttributes`, `backgroundImage` и т.д.

С помощью `UINavigationBarAppearance` можно централизованно и декларативно задавать внешний вид навигационной панели для всего приложения или отдельных экранов.

### Основные свойства UINavigationBarAppearance (самые используемые в 2026)

| Свойство                                      | Тип / Значение по умолчанию                          | Что настраивает                                           | Самый частый сценарий |
|-----------------------------------------------|-------------------------------------------------------|-----------------------------------------------------------|-----------------------|
| `backgroundColor`                             | `UIColor?`                                            | Цвет фона панели                                          | Светлый/тёмный фон |
| `backgroundImage` / `shadowImage`             | `UIImage?`                                            | Фоновое изображение и тень под панелью                    | Градиент, прозрачность |
| `backgroundEffect`                            | `UIVisualEffect?`                                     | Эффект размытия (blur)                                    | `.systemMaterial` — стеклянный стиль |
| `titleTextAttributes`                         | `[NSAttributedString.Key: Any]?`                      | Атрибуты текста заголовка                                 | Шрифт, цвет, размер |
| `largeTitleTextAttributes`                    | `[NSAttributedString.Key: Any]?`                      | Атрибуты большого заголовка (large title)                 | `.large` стиль |
| `buttonAppearance` (UIBarButtonItemAppearance) | `UIBarButtonItemAppearance`                          | Стиль обычных кнопок (back, done и т.д.)                  | Цвет иконок/текста |
| `doneButtonAppearance`                        | `UIBarButtonItemAppearance`                           | Стиль кнопки «Done»                                       | Выделение акцентом |
| `backButtonAppearance`                        | `UIBarButtonItemAppearance`                           | Стиль кнопки «Назад»                                      | Кастомизация стрелки |
| `standardAppearance` (iOS 13+)                | `UINavigationBarAppearance`                           | Внешний вид при обычном размере (не large)                | Основной стиль |
| `scrollEdgeAppearance` (iOS 13+)              | `UINavigationBarAppearance?`                          | Внешний вид, когда список в самом верху (scroll edge)     | Прозрачная панель при скролле |
| `compactAppearance` (iOS 13+)                 | `UINavigationBarAppearance?`                          | Внешний вид в компактном режиме (landscape iPhone)        | Редко используется |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Глобальная кастомизация + адаптация под scroll edge)

```swift
import UIKit

class AppAppearance {
    
    static func setupGlobalNavigationBarAppearance() {
        let appearance = UINavigationBarAppearance()
        
        // Основной стиль (стандартный режим)
        appearance.configureWithOpaqueBackground()
        appearance.backgroundColor = .systemBackground
        appearance.titleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 17, weight: .semibold),
            .foregroundColor: UIColor.label
        ]
        appearance.largeTitleTextAttributes = [
            .font: UIFont.systemFont(ofSize: 34, weight: .bold),
            .foregroundColor: UIColor.label
        ]
        
        // Прозрачная панель, когда контент в самом верху (scroll edge)
        let scrollEdgeAppearance = UINavigationBarAppearance()
        scrollEdgeAppearance.configureWithTransparentBackground()
        scrollEdgeAppearance.backgroundEffect = UIBlurEffect(style: .systemMaterial)
        scrollEdgeAppearance.titleTextAttributes = appearance.titleTextAttributes
        scrollEdgeAppearance.largeTitleTextAttributes = appearance.largeTitleTextAttributes
        
        // Применяем глобально
        UINavigationBar.appearance().standardAppearance = appearance
        UINavigationBar.appearance().scrollEdgeAppearance = scrollEdgeAppearance
        UINavigationBar.appearance().compactAppearance = appearance  // landscape iPhone
        
        // Кнопки навигации
        let buttonAppearance = UIBarButtonItemAppearance(style: .plain)
        buttonAppearance.normal.titleTextAttributes = [
            .foregroundColor: UIColor.systemBlue,
            .font: UIFont.systemFont(ofSize: 17, weight: .medium)
        ]
        
        appearance.buttonAppearance = buttonAppearance
        appearance.backButtonAppearance = buttonAppearance
        appearance.doneButtonAppearance = buttonAppearance
    }
}

// Вызов один раз при запуске приложения (обычно в AppDelegate или SceneDelegate)
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    AppAppearance.setupGlobalNavigationBarAppearance()
    return true
}
```

### Локальная кастомизация для отдельного экрана

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Локальный стиль только для этого экрана
        let appearance = UINavigationBarAppearance()
        appearance.configureWithTransparentBackground()
        appearance.backgroundEffect = UIBlurEffect(style: .systemUltraThinMaterial)
        appearance.titleTextAttributes = [.foregroundColor: UIColor.white]
        appearance.largeTitleTextAttributes = [.foregroundColor: UIColor.white]
        
        navigationItem.standardAppearance = appearance
        navigationItem.scrollEdgeAppearance = appearance
        navigationItem.compactAppearance = appearance
        
        // Большой заголовок
        navigationItem.largeTitleDisplayMode = .always
        title = "Профиль"
    }
}
```

### Лучшие практики UINavigationBarAppearance в 2026 году

- **Глобально** настраивайте через `UINavigationBar.appearance()` в `didFinishLaunchingWithOptions` или `scene(_:willConnectTo:)`  
- **Обязательно** задавайте `scrollEdgeAppearance` — иначе при скролле панель может стать прозрачной/неожиданной  
- **Используйте** `.configureWithOpaqueBackground()`, `.configureWithTransparentBackground()`, `.configureWithDefaultBackground()` — это удобные методы подготовки  
- **Для тёмной темы** — используйте системные цвета (`.label`, `.systemBackground`) — они автоматически адаптируются  
- **Для large title** — задавайте `largeTitleTextAttributes` и `largeTitleDisplayMode = .always` / `.automatic`  
- **Для SwiftUI** — используйте `.navigationBarTitleDisplayMode(.large)` и `.toolbarBackground(.ultraThinMaterial)` — `UINavigationBarAppearance` нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// Глобальная настройка внешнего вида UINavigationBar (стандартный + scroll edge)
static func setupGlobalNavigationBarAppearance() {
    let appearance = UINavigationBarAppearance()
    // ...
}
```

**Короткий итог 2026**:
> `UINavigationBarAppearance` — объект конфигурации внешнего вида **навигационной панели**.  
> В 2026 году:  
> - ключевые свойства — `backgroundColor`, `titleTextAttributes`, `scrollEdgeAppearance`  
> - глобально настраивается через `UINavigationBar.appearance()`  
> - локально — через `navigationItem.standardAppearance` / `scrollEdgeAppearance`  
> - это **единственный современный** способ кастомизировать navigation bar в UIKit  

Удачи с красивой и согласованной навигацией в твоём приложении! 📱