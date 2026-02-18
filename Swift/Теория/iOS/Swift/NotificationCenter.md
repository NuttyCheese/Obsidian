**`NotificationCenter`** — это центральный объект в [[iOS]]/macOS для **передачи уведомлений** (broadcast сообщений) между частями приложения без прямой связи между отправителем и получателем.

Это классический **[[Publisher]]-[[Subscriber]]** (издатель-подписчик) паттерн, встроенный в Foundation.

> Проще говоря: NotificationCenter — это «внутренняя почта приложения»: один объект отправляет письмо, все подписанные на эту тему его получают.

### 1. Когда и зачем использовать NotificationCenter в 2025–2026

| Сценарий (реальный 2026)                    | Почему NotificationCenter здесь идеален                        | Альтернатива (когда лучше не использовать)        |
| ------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------- |
| Изменение темы приложения (light/dark mode) | `UITraitCollection` рассылает уведомление всем                 | [[Combine]] / @Environment в [[SwiftUI]]          |
| Клавиатура появилась/скрылась               | `UIKeyboardWillShow` / `UIKeyboardDidHide`                     | `.keyboardAdaptive()` в SwiftUI                   |
| Пользователь вошёл/вышел из аккаунта        | Разные модули (корзина, профиль, настройки) реагируют          | AuthenticationManager с @Published / async stream |
| Приложение стало активным / ушло в фон      | `UIApplication.didBecomeActiveNotification`                    | ScenePhase в SwiftUI                              |
| Синхронизация данных между экранами         | [[Core Data]] / [[Realm]] отправляет уведомления об изменениях | Combine / Observation / @Observable               |
| Отладка / временные события                 | Быстрое прототипирование без сильной связи                     | —                                                 |

**Важно 2026**:  
NotificationCenter всё ещё жив и используется в **UIKit-проектах** и legacy-коде, но в **новом чистом SwiftUI-проекте** его почти полностью вытеснили:
- `@Environment`
- `@Published` / `@Observable`
- Combine / AsyncStream
- Observation framework (Swift 5.9+)

### 2. Основные методы NotificationCenter (самые используемые)

| Метод / Свойство                           | Что делает                                             | Современный стиль 2026 (рекомендация)      |
| ------------------------------------------ | ------------------------------------------------------ | ------------------------------------------ |
| `addObserver(forName:object:queue:using:)` | Подписка через closure (самый безопасный и популярный) | **Основной способ**                        |
| `addObserver(_:selector:name:object:)`     | Подписка через #selector (старый стиль)                | Использовать только в legacy-коде          |
| `removeObserver(_:)`                       | Удаление подписки по объекту                           | Вызывать в [[deinit]]                      |
| `post(name:object:userInfo:)`              | Отправка уведомления                                   | —                                          |
| `Notification.Name` extension              | Типобезопасные имена уведомлений                       | **Обязательно** для всех своих уведомлений |

### 3. Самый современный и безопасный паттерн 2026 ([[closure]] + token)

```swift
extension Notification.Name {
    static let userDidLogin = Notification.Name("userDidLogin")
    static let cartDidUpdate = Notification.Name("cartDidUpdate")
}

final class CartViewModel {
    private var loginObserver: Any?
    
    init() {
        // Подписка через closure + токен
        loginObserver = NotificationCenter.default.addObserver(
            forName: .userDidLogin,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            guard let self else { return }
            if let userInfo = notification.userInfo,
               let userId = userInfo["userId"] as? String {
                self.loadCart(for: userId)
            }
        }
    }
    
    deinit {
        // Автоматическое удаление подписки (самое безопасное)
        if let token = loginObserver {
            NotificationCenter.default.removeObserver(token)
        }
    }
    
    private func loadCart(for userId: String) {
        // ...
    }
}
```

### 4. Лучшие практики NotificationCenter в 2026

- **Всегда** используй **closure-подписку** (`addObserver(forName:queue:using:)`) — это безопаснее и современнее  
- **Всегда** сохраняй токен (`Any?`) и удаляй подписку в `deinit`  
- **Всегда** используй **`.main` очередь** для UI-обновлений  
- **Всегда** расширяй `Notification.Name` для своих уведомлений — это типобезопасно  
- **Не используй** NotificationCenter для передачи сложных данных — лучше `Combine`, `@Observable`, `AsyncStream`  
- **Не злоупотребляй** — NotificationCenter легко превращается в спагетти-код  
- **Swift 6 strict concurrency** — используй `NotificationCenter.default` осторожно (он не actor-isolated), лучше оборачивать в `@MainActor`  

**Короткий девиз 2026**:
> NotificationCenter — это **почта внутри приложения**: один отправил — все подписанные получили.  
> В 2026 году:  
> - используй **closure + токен** (addObserver(forName:using:))  
> - удаляй подписку в `deinit`  
> - имена уведомлений — через `Notification.Name` extension  
> - для нового кода → **Combine / Observation / @Observable**  
> - NotificationCenter оставь для UIKit legacy и системных уведомлений (клавиатура, ориентация, background/foreground)
