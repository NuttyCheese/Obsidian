**UIHostingController** — это специальный класс в **[[UIKit]]**, который позволяет **встраивать [[SwiftUI]]-представления** (любые `View`) внутрь классической UIKit-иерархии.

Это **основной и единственный официальный мост** между SwiftUI и UIKit в обе стороны.

### Основные сценарии использования UIHostingController (2025–2026)

| Сценарий                                                     | Почему именно UIHostingController                         | Альтернатива (почти нет)         |
| ------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------- |
| Встраивание SwiftUI в старый UIKit-приложение                | Самый безопасный и поддерживаемый способ                  | Нет альтернативы                 |
| Использование SwiftUI в [[UIViewController]]                 | Полный доступ к SwiftUI внутри UIKit-контроллера          | —                                |
| Замена `UIViewController` на SwiftUI                         | `UIHostingController` становится корневым контроллером    | —                                |
| SwiftUI в [[UITabBarController]], [[UINavigationController]] | Легко интегрируется как обычный контроллер                | —                                |
| SwiftUI в ячейке таблицы / коллекции                         | Через `UIHostingConfiguration` (iOS 16+) или вручную      | `UIHostingConfiguration` (новее) |
| Показ модального SwiftUI из UIKit                            | `present(UIHostingController(rootView: MySwiftUIView()))` | —                                |

### Самый популярный и рекомендуемый паттерн (2026)

#### 1. Простое встраивание SwiftUI в UIKit

```swift
import SwiftUI
import UIKit

// 1. Создаём SwiftUI View
struct WelcomeView: View {
    var body: some View {
        VStack(spacing: 24) {
            Text("Добро пожаловать!")
                .font(.largeTitle.bold())
            Button("Начать") {
                // действие
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}

// 2. Оборачиваем в UIHostingController
class WelcomeHostingController: UIHostingController<WelcomeView> {
    
    init() {
        super.init(rootView: WelcomeView())
        
        // Настройки, которые нельзя задать в SwiftUI
        modalPresentationStyle = .fullScreen
        modalTransitionStyle = .crossDissolve
    }
    
    @MainActor required dynamic init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

// 3. Показываем как обычный контроллер
let hostingVC = WelcomeHostingController()
present(hostingVC, animated: true)
```

#### 2. UIHostingController как корневой контроллер приложения

```swift
// В SceneDelegate или AppDelegate (если без сцен)
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    
    let window = UIWindow(windowScene: windowScene)
    
    // Корневой контроллер — SwiftUI
    window.rootViewController = UIHostingController(rootView: ContentView())
    window.makeKeyAndVisible()
    
    self.window = window
}
```

#### 3. Самый современный способ (iOS 16+) — UIHostingConfiguration

Если нужно встроить **небольшой кусок SwiftUI** в ячейку таблицы или коллекции:

```swift
// iOS 16+
let config = UIHostingConfiguration {
    VStack {
        Text("SwiftUI внутри ячейки")
            .font(.headline)
        Image(systemName: "star.fill")
            .foregroundStyle(.yellow)
    }
    .padding()
}
.cellConfigurationHandler = { cell in
    cell.contentConfiguration = config
}
```

### Жизненный цикл UIHostingController

| Метод / Событие                              | Когда вызывается            | Особенности в 2026                         |
| -------------------------------------------- | --------------------------- | ------------------------------------------ |
| `init(rootView:)`                            | Создание контроллера        | Передаём SwiftUI View                      |
| [[viewDidLoad]]`()`                          | После загрузки UIKit-view   | Можно настроить navigationItem, tabBarItem |
| [[viewWillAppear]] / [[viewDidAppear]]       | Стандартный UIKit-цикл      | SwiftUI `.onAppear` вызывается здесь       |
| [[traitCollectionDidChange]]                 | Изменение темы / size class | SwiftUI автоматически адаптируется         |
| [[viewWillDisappear]] / [[viewDidDisappear]] | Уход с экрана               | SwiftUI `.onDisappear` вызывается здесь    |

### Лучшие практики UIHostingController в 2026 году

- **Используйте** `UIHostingController` как **корневой контроллер** — это самый чистый способ перейти на SwiftUI  
- **Передавайте** состояние через `@ObservedObject`, `@StateObject`, `@EnvironmentObject` — избегайте сильных ссылок  
- **Для навигации** — используйте `NavigationStack` / `NavigationView` внутри SwiftUI, а не UIKit UINavigationController  
- **Для таббара** — лучше `TabView` в SwiftUI, но если нужен UIKit UITabBarController — встраивайте HostingController'ы как дочерние  
- **Для производительности** — избегайте слишком глубокой вложенности UIKit → SwiftUI → UIKit  
- **Для ячеек** — iOS 16+ → `UIHostingConfiguration` (быстрее и проще, чем полноценный UIHostingController в ячейке)  
- **Документируйте** — пишите комментарий «UIHostingController — обёртка над SwiftUI WelcomeView, используется как модальный экран»

**Короткий итог 2026**:
> UIHostingController — это **мост** UIKit → SwiftUI.  
> В 2026 году:  
> - основной способ встроить SwiftUI в UIKit-приложение  
> - создаётся через `UIHostingController(rootView: MySwiftUIView())`  
> - полностью поддерживает SwiftUI-анимации, состояние, навигацию  
> - для ячеек — используйте `UIHostingConfiguration` (iOS 16+)  
> Это **единственный официальный** и **самый стабильный** способ комбинировать UIKit и SwiftUI в одном приложении.
