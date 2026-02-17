**Application Life Cycle** (жизненный цикл приложения) в [[iOS]] — это последовательность **состояний**, через которые проходит приложение от момента запуска до его завершения или принудительного удаления из памяти.

Он определяет, **когда** и **как** приложение может выполнять код, сохранять данные, реагировать на системные события и освобождать ресурсы.

### Основные состояния приложения (2026 актуально)

| Состояние          | Когда происходит                                      | Может выполнять код? | Что обычно делают в этом состоянии                          |
|---------------------|--------------------------------------------------------|-----------------------|-------------------------------------------------------------|
| **Not Running**     | Приложение не запущено или было принудительно завершено | Нет                   | Ничего (приложение «мертво»)                                |
| **Inactive**        | Запущено, но **не в фокусе** (входящий звонок, уведомление, переход между экранами) | Да (очень коротко)    | Приостановить анимации, игры, видео                         |
| **Active**          | Приложение на экране, пользователь взаимодействует      | Да (полноценно)       | Основная работа: UI, сеть, анимации, звук                   |
| **Background**      | Приложение ушло в фон (Home, другое приложение впереди) | Да (ограниченное время ~30 сек) | Сохранение данных, завершение задач, отправка локальных уведомлений |
| **Suspended**       | Система заморозила приложение в фоне (нет CPU)         | Нет                   | Ничего (может быть убито системой в любой момент)          |

### Переходы между состояниями (самая частая последовательность)

1. **Not Running → Inactive**  
   Пользователь тапнул по иконке → `didFinishLaunchingWithOptions`

2. **Inactive → Active**  
   Экран полностью загрузился → `applicationDidBecomeActive`

3. **Active → Inactive**  
   Входящий звонок / уведомление / переход между экранами → `applicationWillResignActive`

4. **Inactive → Background**  
   Нажата кнопка Home / другое приложение впереди → `applicationDidEnterBackground`

5. **Background → Suspended**  
   Прошло ~30 секунд фоновой работы → система замораживает приложение

6. **Background / Suspended → Active**  
   Пользователь вернулся в приложение → `applicationWillEnterForeground` → `applicationDidBecomeActive`

7. **Suspended → Not Running**  
   Система убила приложение из-за нехватки памяти → `applicationWillTerminate` (редко вызывается)

### Ключевые методы в AppDelegate / SceneDelegate (2026)

| Метод / Событие                                 | Когда вызывается                               | Где находится в 2026 году (UIKit) | Что обычно делают                                                |
| ----------------------------------------------- | ---------------------------------------------- | --------------------------------- | ---------------------------------------------------------------- |
| `application(_:didFinishLaunchingWithOptions:)` | Самый первый запуск приложения                 | [[AppDelegate]]                   | Инициализация Firebase, Amplitude, Crashlytics, регистрация push |
| `applicationDidBecomeActive(_:)`                | Приложение стало видимым и активным            | AppDelegate / [[SceneDelegate]]   | Запуск анимаций, проверка токенов, обновление виджетов           |
| `applicationWillResignActive(_:)`               | Приложение теряет фокус (уведомление, звонок)  | AppDelegate / SceneDelegate       | Пауза игр, видео, таймеров                                       |
| `applicationDidEnterBackground(_:)`             | Приложение ушло в фон                          | AppDelegate / SceneDelegate       | Сохранение состояния, остановка задач, отправка аналитики        |
| `applicationWillEnterForeground(_:)`            | Приложение возвращается из фона                | AppDelegate / SceneDelegate       | Обновление данных, проверка авторизации                          |
| `applicationWillTerminate(_:)`                  | Приложение завершает работу (редко вызывается) | AppDelegate                       | Финальное сохранение (почти никогда не полагайтесь)              |

### Современный AppDelegate / SceneDelegate в 2026 (минимальный + полезный)

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Инициализация аналитики и сервисов
        FirebaseApp.configure()
        Amplitude.instance().initialize(apiKey: "...")
        
        // Push-уведомления
        UNUserNotificationCenter.current().delegate = self
        application.registerForRemoteNotifications()
        
        print("🚀 Запуск приложения")
        return true
    }
    
    func applicationDidBecomeActive(_ application: UIApplication) {
        print("✅ Активно")
        // Обновить виджеты, проверить токены
    }
    
    func applicationDidEnterBackground(_ application: UIApplication) {
        print("⏸ В фоне")
        // Сохранить состояние, остановить воспроизведение
    }
}

// MARK: - Push-уведомления
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        // Показать уведомление даже когда приложение активно
        completionHandler([.banner, .sound])
    }
}
```

### Короткий девиз 2026

> Application Life Cycle — это когда приложение «рождается», «живёт», «засыпает», «просыпается» и «умирает».  
> В 2026 году AppDelegate всё ещё **обязателен** для глобальной инициализации (push, аналитика, сервисы), но большая часть управления сценами и окнами ушла в **SceneDelegate** и **[[SwiftUI]] @main**.  
> Главное — правильно обрабатывать `didFinishLaunching`, `DidBecomeActive`, `DidEnterBackground` и push.
