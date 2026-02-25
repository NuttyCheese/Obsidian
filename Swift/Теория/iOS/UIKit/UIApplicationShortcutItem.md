**UIApplicationShortcutItem** — это объект, который позволяет добавлять **быстрые действия** (Quick Actions / 3D Touch / Haptic Touch / Long Press) на иконку приложения на домашнем экране iOS.

С помощью этих элементов пользователь может сразу перейти к определённому экрану или выполнить действие, не открывая приложение полностью.

### Актуальность в 2026 году

- Поддерживается на всех современных iPhone (с iOS 9 по настоящее время)  
- На устройствах без 3D Touch (с iPhone 8 и новее) — активируется долгим нажатием  
- На iPad — тоже работает через долгое нажатие на иконку  
- Остаётся **одним из самых эффективных** способов улучшить UX и повысить вовлечённость

### Основные типы и свойства

| Свойство / Тип                            | Описание                                                                 | Обязательно? | Пример |
|-------------------------------------------|--------------------------------------------------------------------------|--------------|--------|
| `type`                                    | Уникальный строковый идентификатор действия (reverse-DNS стиль)          | Да           | `com.example.app.new-note` |
| `localizedTitle`                          | Отображаемое название пункта меню                                        | Да           | "Новая заметка" |
| `localizedSubtitle`                       | Дополнительный текст (подзаголовок)                                      | Нет          | "Создать быстро" |
| `iconType` / `iconFilePath`               | Иконка (системная или кастомная)                                         | Нет          | `.add`, `.compose`, кастомный PNG |
| `userInfo`                                | `[String: NSSecureCoding]` — произвольные данные для обработки           | Нет          | `["noteId": 123]` |
| `UIApplicationShortcutIconType`           | Готовые системные иконки (add, compose, play, pause, bookmark и т.д.)   | Нет          | `.add`, `.search` |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Статические Quick Actions (в Info.plist)

Самый простой и рекомендуемый способ для большинства приложений.

```xml
<!-- Info.plist -->
<key>UIApplicationShortcutItems</key>
<array>
    <dict>
        <key>UIApplicationShortcutItemType</key>
        <string>$(PRODUCT_BUNDLE_IDENTIFIER).new-note</string>
        <key>UIApplicationShortcutItemTitle</key>
        <string>Новая заметка</string>
        <key>UIApplicationShortcutItemSubtitle</key>
        <string>Быстрое создание</string>
        <key>UIApplicationShortcutItemIconType</key>
        <string>UIApplicationShortcutIconTypeCompose</string>
    </dict>
    <dict>
        <key>UIApplicationShortcutItemType</key>
        <string>$(PRODUCT_BUNDLE_IDENTIFIER).search</string>
        <key>UIApplicationShortcutItemTitle</key>
        <string>Поиск</string>
        <key>UIApplicationShortcutItemIconType</key>
        <string>UIApplicationShortcutIconTypeSearch</string>
    </dict>
</array>
```

#### 2. Динамические Quick Actions (в коде)

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, 
                     configurationForConnecting connectingSceneSession: UISceneSession,
                     options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        
        // Динамические действия (добавляются при запуске)
        let shortcutItems: [UIApplicationShortcutItem] = [
            UIApplicationShortcutItem(type: "\(Bundle.main.bundleIdentifier ?? "").new-task",
                                      localizedTitle: "Новая задача",
                                      localizedSubtitle: "Создать быстро",
                                      icon: .type(.add)),
            
            UIApplicationShortcutItem(type: "\(Bundle.main.bundleIdentifier ?? "").scan",
                                      localizedTitle: "Сканировать документ",
                                      icon: .type(.camera))
        ]
        
        application.shortcutItems = shortcutItems
        
        let config = UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
        config.delegateClass = SceneDelegate.self
        return config
    }
}
```

#### 3. Обработка нажатия (в SceneDelegate или AppDelegate)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    func windowScene(_ windowScene: UIWindowScene, 
                     performActionFor shortcutItem: UIApplicationShortcutItem,
                     completionHandler: @escaping (Bool) -> Void) {
        
        let bundleID = Bundle.main.bundleIdentifier ?? ""
        
        switch shortcutItem.type {
        case "\(bundleID).new-note":
            // Переход на экран создания заметки
            if let nav = window?.rootViewController as? UINavigationController,
               let root = nav.viewControllers.first as? MainViewController {
                root.createNewNote()
                completionHandler(true)
            }
            
        case "\(bundleID).search":
            // Открываем поиск
            if let root = window?.rootViewController as? MainTabBarController {
                root.selectedIndex = 1 // вкладка поиска
                completionHandler(true)
            }
            
        default:
            completionHandler(false)
        }
    }
}
```

### Лучшие практики UIApplicationShortcutItem в 2026 году

- **Используйте** статические в Info.plist — проще, быстрее, не требует кода  
- **Динамические** — добавляйте только если они зависят от состояния (например, "Продолжить просмотр последнего фильма")  
- **Тип** — всегда в формате `com.company.app.action` (reverse-DNS)  
- **Иконки** — предпочитайте системные (`UIApplicationShortcutIconType`) — они выглядят нативно  
- **Количество** — не больше 4–5 (Apple рекомендует не перегружать меню)  
- **Для SwiftUI** — используйте `.onContinueUserActivity` в сцене — UIApplicationShortcutItem нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// Динамические Quick Actions при запуске приложения
private func setupDynamicShortcuts() {
    let bundleID = Bundle.main.bundleIdentifier ?? ""
    UIApplication.shared.shortcutItems = [
        UIApplicationShortcutItem(type: "\(bundleID).new-note",
                                  localizedTitle: "Новая заметка",
                                  icon: .type(.compose))
    ]
}
```

**Короткий итог 2026**:
> `UIApplicationShortcutItem` — элемент **быстрого действия** на иконке приложения (Quick Actions).  
> В 2026 году:  
> - статические — в Info.plist (самый простой способ)  
> - динамические — через `UIApplication.shared.shortcutItems`  
> - обработка — в `windowScene(_:performActionFor:completionHandler:)`  
> - идеален для "Новая заметка", "Поиск", "Сканировать QR", "Продолжить просмотр"  
> - это **мощный** инструмент для улучшения UX и повышения вовлечённости  

Удачи с удобными и быстрыми действиями прямо с домашнего экрана в твоём приложении! 🚀