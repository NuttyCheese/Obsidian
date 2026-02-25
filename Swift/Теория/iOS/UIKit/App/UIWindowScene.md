**UIWindowScene** — это класс, представляющий **экземпляр сцены** (scene) в [[iOS]]-приложении, начиная с iOS 13 (2019 год), когда Apple ввела **[[UIScene]]**-архитектуру.

Он заменил старое понятие «одно окно на приложение» на **множественные сцены** (множество окон), что особенно важно на iPad и в macOS Catalyst, но также используется на iPhone.

### Основные моменты о UIWindowScene (актуально на 2026 год)

| Характеристика                  | Описание                                                       | Почему важно в 2026                                                           |
| ------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Представляет одно окно/сцену    | Каждое окно приложения — это отдельная `UIWindowScene`         | Поддержка Split View, Slide Over, Stage Manager (iPad), несколько окон на Mac |
| Имеет собственный [[UIWindow]]  | `scene.windows` содержит все окна сцены (обычно одно)          | Управление окнами внутри сцены                                                |
| Управляется `UISceneSession`    | Система создаёт сессию → сцена → окно                          | Жизненный цикл сцены                                                          |
| Связан с `UIWindowSceneSession` | Содержит конфигурацию (роль, параметры запуска)                | Восстановление состояния при многозадачности                                  |
| Поддерживает внешние дисплеи    | С iPadOS 13+ и macOS Catalyst можно подключать внешний монитор | Приложения с расширенным экраном                                              |
| Имеет `interfaceOrientation`    | Текущая ориентация сцены (не устройства!)                      | Корректная работа в Split View                                                |

### Основные свойства и методы UIWindowScene

| Свойство / Метод                   | Тип / Возвращает                                   | Что делает / зачем нужен                              | Самый частый сценарий |
|------------------------------------|-----------------------------------------------------|-------------------------------------------------------|-----------------------|
| `windows`                          | `[UIWindow]`                                        | Все окна, принадлежащие этой сцене                    | Доступ к основному окну |
| `keyWindow` (iOS 15+)              | `UIWindow?`                                         | Главное окно сцены (то, что получает фокус ввода)     | Показ алертов, получение rootViewController |
| `interfaceOrientation`             | `UIInterfaceOrientation`                            | Текущая ориентация сцены                              | Адаптация UI под Split View |
| `coordinateSpace`                  | `UICoordinateSpace`                                 | Координатная система сцены                            | Конвертация координат между окнами |
| `screen`                           | `UIScreen`                                          | Экран, на котором отображается сцена                  | Поддержка внешних дисплеев |
| `titlebar` (macOS Catalyst)        | `UITitlebar?`                                       | Управление заголовком окна на Mac                     | Кастомизация заголовка |
| `requestGeometryUpdate(_:errorHandler:)` | `Void` (iOS 16+)                             | Запрос изменения размера/позиции окна                 | Адаптация под Stage Manager |
| `activationState`                  | `UIScene.ActivationState`                           | Текущее состояние сцены (foreground, background и т.д.) | Реакция на смену состояния |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Получение текущей сцены и окна (самый частый код)

```swift
extension UIViewController {
    var currentWindowScene: UIWindowScene? {
        view.window?.windowScene
    }
    
    var currentKeyWindow: UIWindow? {
        currentWindowScene?.keyWindow
    }
    
    var currentRootViewController: UIViewController? {
        currentKeyWindow?.rootViewController
    }
}

// Использование
if let scene = currentWindowScene {
    print("Ориентация сцены:", scene.interfaceOrientation)
}
```

#### 2. Создание и настройка новой сцены (редко нужно вручную)

```swift
// В Info.plist указываем поддержку сцен
// UIApplicationSceneManifest → UIApplicationSupportsMultipleScenes = YES

// Создание новой сцены (например, для внешнего дисплея)
let sceneRequest = UISceneSessionActivationRequest.newSceneSession(
    role: .windowApplication,
    configuration: .default
)

UIApplication.shared.requestSceneSessionActivation(
    nil,
    userActivity: nil,
    options: nil
) { error in
    if let error {
        print("Ошибка создания сцены:", error)
    }
}
```

#### 3. Работа с внешним дисплеем (iPad + Stage Manager)

```swift
NotificationCenter.default.addObserver(
    forName: UIScreen.didConnectNotification,
    object: nil,
    queue: .main
) { notification in
    guard let screen = notification.object as? UIScreen else { return }
    
    let scene = UIWindowScene(session: .init(), connectionOptions: .init())
    let window = UIWindow(windowScene: scene)
    window.screen = screen
    window.rootViewController = ExternalDisplayViewController()
    window.makeKeyAndVisible()
}
```

### Лучшие практики UIWindowScene в 2026 году

- **Всегда** получайте окно/сцену через `view.window?.windowScene` или `UIApplication.shared.connectedScenes`  
- **Для keyWindow** — используйте `windowScene.keyWindow` (iOS 15+) вместо устаревшего `UIApplication.shared.keyWindow`  
- **Для ориентации** — используйте `windowScene.interfaceOrientation`, а не `UIDevice.current.orientation`  
- **Для нескольких окон** — проверяйте `UIApplication.shared.connectedScenes` и фильтруйте по `UIWindowScene`  
- **Для внешних дисплеев** — слушайте `UIScreen.didConnectNotification` и `UIScreen.didDisconnectNotification`  
- **В SwiftUI** — используйте `@Environment(\.scenePhase)` и `.onChange(of: scenePhase)` — `UIWindowScene` нужен редко  
- **Документируйте** — пишите комментарий:

```swift
/// Возвращает активную UIWindowScene текущего контроллера
var currentWindowScene: UIWindowScene? {
    view.window?.windowScene
}
```

**Короткий итог 2026**:
> `UIWindowScene` — это **объект сцены** (окна) в iOS 13+ архитектуре.  
> В 2026 году:  
> - каждое окно = отдельная сцена  
> - ключевые свойства — `windows`, `keyWindow`, `interfaceOrientation`, `screen`  
> - используйте `view.window?.windowScene` для доступа к текущей сцене  
> - важно для Split View, Stage Manager, внешних дисплеев и многозадачности  
> - это **основа** современной многоконной архитектуры iPad и Mac Catalyst  
