**OSLog** — это современная система логирования Apple, представленная в [[iOS]] 10 / macOS 10.12 (2016 год) и ставшая **рекомендуемым стандартом** для всего логирования в приложениях Apple к 2026 году.

Она полностью заменила устаревший `print()`, `NSLog()`, `os_log` (старый C-[[API]]) и даже многие сторонние логгеры (CocoaLumberjack, SwiftyBeaver и т.д.) в новых проектах.

### Почему OSLog — лучший выбор в 2026 году

| Преимущество                        | Почему это критично в 2026 году                                                                                 |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Полная интеграция с Console.app** | Логи видны в реальном времени на устройстве и в [[Xcode]] без дополнительных инструментов                       |
| **Контекст и метаданные**           | Автоматически добавляет timestamp, process ID, thread ID, subsystem, category                                   |
| **Уровни логирования**              | `.debug`, `.info`, `.notice`, `.error`, `.fault` — с разной детализацией и приватностью                         |
| **Приватность данных**              | Поддержка **redaction** (`%{public}@`, `%{private}@`, `%{sensitive}@`) — скрывает чувствительные данные в логах |
| **Производительность**              | Очень низкие накладные расходы (быстрее `print()` в 10–100 раз)                                                 |
| **Хранение и ротация**              | Логи сохраняются в системе (до 500 МБ на процесс), ротация автоматическая                                       |
| **[[Swift Concurrency]]**           | Полностью thread-safe и [[Sendable]]-safe                                                                       |
| **Xcode 16+ / Instruments**         | Полная поддержка: фильтры, поиск, экспорт, осмотр в реальном времени                                            |
| **Облачная аналитика**              | Интеграция с MetricKit, OSLogStore, Signposts для продвинутой телеметрии                                        |

### Уровни логирования OSLog (от самого детального к критическому)

| Уровень       | Когда использовать                              | Видимость в Console.app | Приватность по умолчанию | Пример использования |
|---------------|-------------------------------------------------|---------------------------|---------------------------|-----------------------|
| `.debug`      | Очень детальная отладка (только в debug-билде)  | Только при включённом debug-режиме | Высокая                   | Временные отладочные логи |
| `.info`       | Информационные события, нормальная работа       | Да                        | Средняя                   | Успешная загрузка данных |
| `.notice`     | Важные события, но не ошибки                    | Да                        | Средняя                   | Пользователь вошёл в аккаунт |
| `.error`      | Ошибки, которые можно обработать                | Да                        | Высокая                   | Сетевая ошибка, retry |
| `.fault`      | Критические сбои, краши, повреждение данных     | Да (всегда)               | Высокая                   | Невосстановимая ошибка |

### Самый современный и рекомендуемый паттерн OSLog в Swift 2026

```swift
import os.log

// 1. Создаём логгер один раз (лучше в глобальном или статическом месте)
extension Logger {
    static let network = Logger(subsystem: Bundle.main.bundleIdentifier ?? "com.example.app", category: "Network")
    static let ui      = Logger(subsystem: Bundle.main.bundleIdentifier ?? "com.example.app", category: "UI")
    static let auth    = Logger(subsystem: Bundle.main.bundleIdentifier ?? "com.example.app", category: "Auth")
}

// 2. Использование в коде
@MainActor
class AuthViewModel: ObservableObject {
    
    func login(email: String, password: String) async throws {
        Logger.auth.info("Попытка входа для пользователя: \(email, privacy: .public)")
        
        do {
            let token = try await api.login(email: email, password: password)
            Logger.auth.notice("Успешный вход: токен получен")
            
        } catch {
            Logger.auth.error("Ошибка входа: \(error.localizedDescription, privacy: .public)")
            throw error
        }
    }
    
    func fetchProfile() async {
        Logger.network.debug("Запрос профиля начат")
        
        do {
            let profile = try await api.fetchProfile()
            Logger.network.info("Профиль загружен: \(profile.name, privacy: .public)")
        } catch {
            Logger.network.fault("Критическая ошибка при загрузке профиля: \(error.localizedDescription, privacy: .public)")
        }
    }
}
```

### Лучшие практики OSLog в Swift 2026

- **Создавай Logger один раз** — используй `static let` в extension или глобально  
- **Subsystem** — обычно `Bundle.main.bundleIdentifier` (или reverse-DNS)  
- **Category** — короткие, понятные строки: "Network", "UI", "Auth", "Database", "Analytics"  
- **Приватность** — всегда указывай уровень приватности:
  - `%{public}@` — видно всем (email, username)  
  - `%{private}@` — скрыто в консоли (пароль, токен)  
  - `%{sensitive}@` — ещё более скрыто (в продакшен-логах)  
- **Уровни** — `.debug` только в debug-билдах, `.fault` — только для критических сбоев  
- **Swift 6 strict concurrency** — `Logger` полностью Sendable и thread-safe  
- **Не используй `print()` / `debugPrint()`** — они не попадают в Console.app и не имеют приватности  
- **Тестирование** — проверяй логи через Console.app или `OSLogStore` в тестах  
- **Документируйте** — пиши комментарий «OSLog — категория Network, уровень info»

**Короткий девиз 2026**:
> «OSLog в 2026 году — это когда ты хочешь логировать так, чтобы это было быстро, безопасно, приватно и видно в Console.app без лишних инструментов.  
> print() и NSLog() — это уже legacy.  
> Самый важный момент — правильно выбрать subsystem + category + уровень приватности.»
