**SceneDelegate** — это ключевой класс в современных UIKit-приложениях (iOS 13+), который отвечает за **жизненный цикл отдельной сцены** (окна) приложения.

С появлением **многозадачности** на iPad, Split View, Slide Over, Stage Manager, внешних дисплеев и нескольких окон на Mac Catalyst, Apple разделила ответственность:

- **AppDelegate** — управляет **всем приложением** глобально (push-токены, запуск, аналитика, глобальные сервисы)
- **SceneDelegate** — управляет **каждым отдельным окном/сценой** (создание окна, rootViewController, состояния конкретного окна)

Это позволяет приложению иметь **несколько независимых окон** одновременно (например, два разных документа в одном приложении).

### 1. Когда и зачем появился SceneDelegate (короткая история)

- До iOS 13: весь жизненный цикл был в **AppDelegate**  
- iOS 13 (2019): Apple ввела **UIScene** и **UIWindowSceneDelegate**  
- С тех пор почти все новые UIKit-проекты используют **Scene-based lifecycle**  
- В 2026 году (iOS 19+) это **стандарт** для всех приложений, поддерживающих несколько сцен

Если ты создаёшь новый проект в Xcode 17+ → он по умолчанию использует **SceneDelegate** + `@main`.

### 2. Основные состояния сцены (и методы SceneDelegate)

| Состояние сцены            | Когда происходит                                      | Метод в SceneDelegate                              | Что обычно делают здесь |
|----------------------------|-------------------------------------------------------|----------------------------------------------------|-------------------------|
| **Not Connected**          | Сцена ещё не создана                                  | —                                                  | — |
| **Foreground Inactive**    | Сцена создана, но не в фокусе (например, при запуске) | `scene(_:willConnectTo:options:)`                  | Создание окна, rootViewController, настройка UI |
| **Foreground Active**      | Сцена на экране, пользователь взаимодействует         | `sceneDidBecomeActive(_:)`                         | Запуск анимаций, обновление данных, проверка токенов |
| **Background**             | Сцена ушла в фон (другое окно впереди)                | `sceneDidEnterBackground(_:)`                      | Сохранение состояния сцены, остановка таймеров |
| **Suspended**              | Система заморозила сцену (нет CPU)                    | —                                                  | Ничего (может быть убита) |
| **Disconnected**           | Сцена закрыта или удалена системой                    | `sceneDidDisconnect(_:)`                           | Освобождение ресурсов сцены |

### 3. Самый важный метод — `scene(_:willConnectTo:options:)`

Это аналог `didFinishLaunchingWithOptions` из AppDelegate, но **для каждой сцены**.

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    
    // 1. Проверяем, что это именно UIWindowScene
    guard let windowScene = (scene as? UIWindowScene) else { return }
    
    // 2. Создаём окно для этой сцены
    let window = UIWindow(windowScene: windowScene)
    self.window = window
    
    // 3. Настраиваем rootViewController
    let rootVC = MainTabBarController()
    window.rootViewController = rootVC
    
    // 4. Делаем окно видимым и ключевым
    window.makeKeyAndVisible()
    
    // 5. (Опционально) обрабатываем launch options для этой сцены
    if let shortcutItem = connectionOptions.shortcutItem {
        handleShortcut(shortcutItem)
    }
}
```

### 4. Полный современный SceneDelegate 2026 (минимальный + полезный)

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    // Самый важный метод — создание сцены
    func scene(_ scene: UIScene,
               willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(windowScene: windowScene)
        self.window = window
        
        // Настраиваем корневой контроллер (чаще всего TabBar или Navigation)
        let rootVC = MainTabBarController()
        window.rootViewController = rootVC
        
        // Делаем окно видимым
        window.makeKeyAndVisible()
        
        // Обрабатываем быстрые действия / Universal Links / Shortcuts
        if let shortcut = connectionOptions.shortcutItem {
            handleShortcut(shortcut)
        }
        
        if let urlContexts = connectionOptions.urlContexts.first {
            handleURLContext(urlContexts)
        }
    }
    
    // Сцена стала активной (пользователь видит это окно)
    func sceneDidBecomeActive(_ scene: UIScene) {
        print("Сцена активна")
        // Можно обновить виджеты, проверить токены
    }
    
    // Сцена ушла в фон
    func sceneDidEnterBackground(_ scene: UIScene) {
        print("Сцена в фоне")
        // Сохранить состояние этой сцены
    }
    
    // Сцена будет отключена (закрыта, удалена системой)
    func sceneDidDisconnect(_ scene: UIScene) {
        print("Сцена отключена")
        // Освободить ресурсы этой сцены
        window = nil
    }
    
    // Сцена вернётся на передний план
    func sceneWillEnterForeground(_ scene: UIScene) {
        print("Сцена выходит из фона")
        // Подготовить UI, обновить данные
    }
    
    // Сцена временно неактивна (уведомление, звонок и т.д.)
    func sceneWillResignActive(_ scene: UIScene) {
        print("Сцена временно неактивна")
        // Пауза анимаций, видео
    }
}
```

### 5. Отличия AppDelegate vs SceneDelegate (таблица 2026)

| Характеристика                          | AppDelegate                                      | SceneDelegate                                      |
|-----------------------------------------|--------------------------------------------------|----------------------------------------------------|
| Управляет                                  | Всём приложением глобально                       | Конкретной сценой (окном)                          |
| Сколько экземпляров                        | Один на приложение                               | Один на каждую сцену (может быть несколько)        |
| didFinishLaunchingWithOptions              | Да (глобальный запуск)                           | Нет (только willConnectTo)                         |
| Создание окна                              | Старый способ (до iOS 13)                        | window = UIWindow(windowScene:)                    |
| Поддержка нескольких окон                  | Ограниченная                                     | Полная (Split View, Stage Manager, внешний дисплей)|
| Push-токены, глобальные уведомления         | Да                                               | Нет (регистрирует AppDelegate)                     |
| Состояния сцены (active, background)       | Глобальные                                       | Локальные для каждой сцены                         |

### 6. Как проверить, использует ли проект SceneDelegate

В новом проекте Xcode 17+ → **да**, по умолчанию:

- Info.plist имеет ключ `UIApplicationSceneManifest`
- Есть класс `SceneDelegate`
- `@main` стоит над AppDelegate, но сцены включены

Если в Info.plist **нет** `UIApplicationSceneManifest` → проект использует старый lifecycle (только AppDelegate).

### Короткий девиз 2026

> SceneDelegate — это «мозг» **каждого отдельного окна** приложения.  
> AppDelegate — «мозг» всего приложения.  
> В 2026 году в UIKit-проектах **SceneDelegate** отвечает за создание окна, rootViewController и состояния каждой сцены.  
> Без него многозадачность, несколько окон и Stage Manager работать не будут.

Удачи с правильным управлением сценами и окнами в твоём приложении! 🪟