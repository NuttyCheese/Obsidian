**Логгирование** — это процесс записи структурированной информации о работе приложения во время выполнения.  
Основные цели:

- Отладка (debugging)  
- Мониторинг в production  
- Анализ производительности  
- Диагностика крашей и ошибок  
- Аудит безопасности  
- Сбор метрик для аналитики

### 1. Уровни логгирования (стандарт 2026)

| Уровень     | Приоритет         | Когда использовать                                       | Пример сообщения                              | В production включать? | В Crashlytics/Sentry |
| ----------- | ----------------- | -------------------------------------------------------- | --------------------------------------------- | ---------------------- | -------------------- |
| **Fault**   | 1 (самый высокий) | Критическая ошибка, приводящая к крашу или потере данных | "Не удалось инициализировать ключ шифрования" | Да (всегда)            | Да (как fatal)       |
| **Error**   | 2                 | Ошибка, которая влияет на функциональность               | "Не удалось загрузить профиль пользователя"   | Да                     | Да                   |
| **Warning** | 3                 | Потенциальная проблема, но приложение продолжает работу  | "[[API]] вернул 429 Too Many Requests"        | Да                     | Иногда (как warning) |
| **Info**    | 4                 | Важные события жизненного цикла                          | "Пользователь вошёл в аккаунт"                | Да (можно фильтровать) | Редко                |
| **Debug**   | 5                 | Детальная информация для отладки                         | "Загружено 42 поста из API"                   | Только в debug-сборках | Нет                  |
| **Trace**   | 6 (самый низкий)  | Очень подробная трассировка (временные метки, шаги)      | "Начало выполнения функции fetchPosts()"      | Только локально        | Нет                  |

**Рекомендация Apple 2026 ([[OSLog]] + Logger)**:  
Используйте **только** `Logger` из `os` (новый API с iOS 14+).  
Старый `print`, `NSLog`, `os_log` устарели и **не рекомендуются**.

### 2. Современный стандарт 2026 — Logger (os.Logger)

```swift
import os.log

// Глобальный логгер (рекомендуется один на модуль/подсистему)
private let logger = Logger(subsystem: "com.example.myapp", category: "network")

// Использование
logger.debug("Начало загрузки списка постов")
logger.info("Загружено \(posts.count) постов")
logger.warning("API вернул 429 — слишком много запросов")
logger.error("Не удалось декодировать ответ: \(error.localizedDescription)")
logger.fault("Критическая ошибка: токен невалиден")
```

#### Уровни в Logger (2026)

| Метод       | Уровень [[OSLog]] | В консоли [[Xcode]] | В production (sysdiagnose) | В Sentry/Crashlytics |
| ----------- | ----------------- | ------------------- | -------------------------- | -------------------- |
| `.trace`    | .debug            | Да                  | Нет                        | Нет                  |
| `.debug`    | .debug            | Да                  | Нет                        | Нет                  |
| `.info`     | .info             | Да                  | Да (редко)                 | Редко                |
| `.notice`   | .notice           | Да                  | Да                         | Иногда               |
| `.warning`  | .error            | Да                  | Да                         | Да                   |
| `.error`    | .error            | Да                  | Да                         | Да                   |
| `.critical` | .fault            | Да                  | Да                         | Да (как fatal)       |

### 3. Полные примеры кода

#### Пример 1 — Рекомендуемая структура логгеров в приложении

```swift
import os.log

enum Log {
    static let general   = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "general")
    static let network   = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "network")
    static let ui        = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "ui")
    static let database  = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "database")
    static let auth      = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "auth")
    static let analytics = Logger(subsystem: Bundle.main.bundleIdentifier ?? "app", category: "analytics")
}

// Использование в разных частях приложения
Log.network.debug("Начало запроса к /users")
Log.auth.info("Пользователь \(userId) успешно вошёл")
Log.ui.warning("Тёмная тема не поддерживается на устройстве")
Log.database.error("Не удалось сохранить пост: \(error.localizedDescription)")
```

#### Пример 2 — Логирование с приватностью (два уровня: [[public]] и [[private]])

```swift
Log.network.info("Запрос к \(url.absoluteString, privacy: .public)")          // видно в production
Log.network.debug("Токен: \(token, privacy: .private)")                      // скрыто в production
Log.auth.error("Ошибка входа для пользователя \(email, privacy: .public(mask: .hash))")
```

**privacy уровни** (2026):

- `.public` — видно всегда  
- `.private` — скрыто в sysdiagnose / Sentry  
- `.private(mask: .hash)` — заменяется на хэш  
- `.sensitive` — полностью скрыто (только в debug)

#### Пример 3 — Логирование ошибок с контекстом

```swift
enum NetworkError: Error, LocalizedError {
    case invalidURL
    case decodingFailed(Error)
    case serverError(statusCode: Int)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Некорректный URL"
        case .decodingFailed(let error): return "Ошибка декодирования: \(error.localizedDescription)"
        case .serverError(let code): return "Ошибка сервера: \(code)"
        }
    }
}

func fetchData<T: Decodable>(from url: URL) async throws -> T {
    Log.network.debug("Начало запроса к \(url.absoluteString, privacy: .public)")
    
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        Log.network.fault("Ответ не является HTTPURLResponse")
        throw URLError(.badServerResponse)
    }
    
    Log.network.info("Получен ответ со статусом \(httpResponse.statusCode, privacy: .public)")
    
    guard (200...299).contains(httpResponse.statusCode) else {
        Log.network.error("Сервер вернул ошибку \(httpResponse.statusCode)")
        throw NetworkError.serverError(statusCode: httpResponse.statusCode)
    }
    
    do {
        let decoded = try JSONDecoder().decode(T.self, from: data)
        Log.network.debug("Успешно декодировано \(T.self, privacy: .public)")
        return decoded
    } catch {
        Log.network.error("Ошибка декодирования: \(error.localizedDescription, privacy: .public)")
        throw NetworkError.decodingFailed(error)
    }
}
```

#### Пример 4 — Интеграция с Crashlytics / Sentry

```swift
import FirebaseCrashlytics
import Sentry

func logError(_ error: Error, level: OSLogType = .error) {
    Log.general.error("\(error.localizedDescription, privacy: .public)")
    
    // Crashlytics
    Crashlytics.crashlytics().record(error: error)
    
    // Sentry
    let sentryEvent = Event()
    sentryEvent.message = SentryMessage(formatted: error.localizedDescription)
    sentryEvent.level = level == .fault ? .fatal : .error
    SentrySDK.capture(event: sentryEvent)
}
```

### 5. Таблица: сравнение систем логгирования 2026

| Система            | Уровни            | Приватность данных | Поддержка async/await | Интеграция с Sentry/Crashlytics | Рекомендация 2026 |
|--------------------|-------------------|---------------------|------------------------|----------------------------------|-------------------|
| `os.Logger`        | trace–critical    | Отличная (public/private) | Полная                 | Да (ручная)                      | Основной выбор    |
| `print()`          | Нет уровней       | Нет                 | Да                     | Нет                              | Только debug      |
| `NSLog`            | Нет уровней       | Нет                 | Да                     | Частичная                        | Устарел           |
| `OSLog` (старый)   | debug–fault       | Хорошая             | Нет                    | Частичная                        | Устарел           |
| SwiftLog (Logging) | trace–critical    | Хорошая             | Полная                 | Да (через handler)               | Для серверов      |

### 6. Лучшие практики 2026 года

- **Всегда** используйте `Logger(subsystem:category:)` — один на модуль/подсистему  
- Уровни:  
  - `.debug` / `.trace` — только в debug-сборках  
  - `.info` / `.notice` — жизненный цикл, важные события  
  - `.warning` / `.error` — потенциальные и реальные проблемы  
  - `.critical` / `.fault` — краши и критические сбои  
- Используйте **privacy уровни** для чувствительных данных  
- Логируйте **контекст** (userId, requestId, screenName)  
- Интегрируйте с **Crashlytics** / **Sentry** / **Datadog**  
- В [[Swift]] 6 — включайте **strict concurrency** и логируйте actor-isolation  
- Для production — фильтруйте `.debug` и `.trace` на уровне сборки

**Короткий девиз**:
> «Логи — это чёрный ящик самолёта.  
> Пиши их на правильном уровне, с приватностью и контекстом — и в любой момент сможешь понять, почему приложение упало или пользователь ушёл.»
