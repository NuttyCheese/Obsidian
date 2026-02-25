**UIScreen** — это класс в [[UIKit]], который представляет **физический экран** устройства (или внешний дисплей), к которому подключено приложение.

В 2026 году UIScreen остаётся **ключевым** для:

- получения характеристик экрана (разрешение, масштаб, яркость, тип дисплея)
- адаптации интерфейса под разные устройства (iPhone, iPad, внешний монитор)
- поддержки **многоконности** и **внешних дисплеев** (iPadOS, Mac Catalyst)
- работы с **Dynamic Island**, **Always-On Display**, **ProMotion** и т.д.

### Основные свойства UIScreen (самые используемые в 2026)

| Свойство                            | Тип / Возвращает         | Что даёт / зачем нужно                                 | Самый частый сценарий                                  |
| ----------------------------------- | ------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| `main`                              | `UIScreen` (статическое) | Главный экран устройства                               | Почти всегда используем `UIScreen.main`                |
| `bounds`                            | [[CGRect]]               | Размер экрана в поинтах (логические пиксели)           | `UIScreen.main.bounds` — размер окна                   |
| `nativeBounds`                      | `CGRect`                 | Физическое разрешение в пикселях (без scale)           | Проверка 4K/Retina                                     |
| `scale`                             | [[CGFloat]] (1x, 2x, 3x) | Масштаб экрана (Retina, Super Retina XDR)              | `UIScreen.main.scale` — для выбора @2x/@3x изображений |
| `nativeScale`                       | `CGFloat`                | Физический масштаб (обычно совпадает с scale)          | Редко                                                  |
| `brightness`                        | `CGFloat` (0.0...1.0)    | Текущая яркость экрана (можно менять)                  | Автоматическая регулировка яркости                     |
| `currentMode`                       | `UIScreenMode?`          | Текущий режим дисплея (разрешение, частота обновления) | ProMotion 120 Гц                                       |
| `maximumFramesPerSecond` (iOS 10+)  | [[Int]]                  | Максимальная частота обновления (60 или 120 Гц)        | Оптимизация анимаций                                   |
| `traitCollection`                   | [[UITraitCollection]]    | Черты экрана (size class, userInterfaceStyle и т.д.)   | Адаптация под тёмную тему / size class                 |
| `snapshotView(afterScreenUpdates:)` | UIView`?`                | Скриншот всего экрана                                  | Анимация переходов, эффекты                            |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Получение текущего экрана и его характеристик

```swift
extension UIViewController {
    var currentScreen: UIScreen? {
        view.window?.screen ?? UIScreen.main
    }
    
    var screenScale: CGFloat {
        currentScreen?.scale ?? UIScreen.main.scale
    }
    
    var screenBoundsInPoints: CGRect {
        currentScreen?.bounds ?? UIScreen.main.bounds
    }
    
    var screenBoundsInPixels: CGRect {
        currentScreen?.nativeBounds ?? UIScreen.main.nativeBounds
    }
    
    var isProMotionDisplay: Bool {
        currentScreen?.maximumFramesPerSecond ?? 60 > 60
    }
    
    var isDarkMode: Bool {
        traitCollection.userInterfaceStyle == .dark
    }
}
```

#### 2. Реакция на подключение/отключение внешнего дисплея

```swift
class ExternalDisplayManager {
    
    init() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(screenDidConnect),
            name: UIScreen.didConnectNotification,
            object: nil
        )
        
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(screenDidDisconnect),
            name: UIScreen.didDisconnectNotification,
            object: nil
        )
    }
    
    @objc private func screenDidConnect(notification: Notification) {
        guard let newScreen = notification.object as? UIScreen else { return }
        
        // Создаём новую сцену для внешнего дисплея
        let sceneRequest = UIWindowSceneSessionActivationRequest.newSceneSession(
            role: .windowExternalDisplay,
            configuration: .default
        )
        
        UIApplication.shared.requestSceneSessionActivation(
            nil,
            userActivity: nil,
            options: nil
        ) { error in
            if let error {
                print("Ошибка создания сцены на внешнем дисплее:", error)
            }
        }
    }
    
    @objc private func screenDidDisconnect(notification: Notification) {
        // Очистка или переключение контента
        print("Внешний дисплей отключён")
    }
}
```

#### 3. Адаптация интерфейса под яркость и ProMotion

```swift
class AdaptiveViewController: UIViewController {
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        // Адаптация под яркость
        if let brightness = UIScreen.main.brightness {
            if brightness < 0.3 {
                // Тёмный режим + низкая яркость → увеличить контраст
                view.backgroundColor = .systemGray6
            }
        }
        
        // Адаптация анимаций под 120 Гц
        if UIScreen.main.maximumFramesPerSecond > 60 {
            // Более плавные анимации
            UIView.animate(withDuration: 0.25) {
                // ...
            }
        }
    }
}
```

### Лучшие практики UIScreen в 2026 году

- **Всегда** используйте `UIScreen.main` или `view.window?.screen` — не полагайтесь на `UIScreen.screens.first`  
- **Для размера** — используйте `bounds` (в поинтах) — это логический размер для Auto Layout  
- **Для ретина** — проверяйте `scale` (2x / 3x) при выборе изображений (@2x/@3x)  
- **Для внешних дисплеев** — слушайте `UIScreen.didConnectNotification` и `didDisconnectNotification`  
- **Для ProMotion** — используйте `maximumFramesPerSecond` для адаптации анимаций (60 → 120 Гц)  
- **Для яркости** — читайте `brightness` и адаптируйте контраст/цвета  
- **Для [[SwiftUI]]** — используйте `@Environment(\.displayScale)`, `@Environment(\.colorScheme)` — UIScreen нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// Текущий экран устройства (основной или внешний)
var currentScreen: UIScreen {
    view.window?.screen ?? UIScreen.main
}
```

**Короткий итог 2026**:
> `UIScreen` — объект, представляющий **физический экран** устройства или внешний дисплей.  
> В 2026 году:  
> - ключевые свойства — `main`, `bounds`, `scale`, `brightness`, `maximumFramesPerSecond`  
> - самый частый доступ — `UIScreen.main.bounds`, `view.window?.screen`  
> - критично для адаптации под iPad, внешние мониторы, ProMotion, Dynamic Island  
> - это **основа** работы с дисплеями и адаптивным интерфейсом в UIKit  
