**URLSessionConfiguration** — это класс в [[Foundation]], который определяет **конфигурацию** для создания объекта [[URLSession]].

Он позволяет настраивать поведение сессии: таймауты, кэширование, заголовки по умолчанию, обработку фоновых задач, ограничения на cellular-сеть и многое другое.

В 2026 году ([[Swift]] 6+, [[iOS]] 18+, macOS 15+) это **единственный способ** тонко настроить сетевые запросы в приложении Apple.

### Три основные предопределённые конфигурации (самые частые)

| Конфигурация  | Когда использовать                                | Ключевые особенности                               | Рекомендация 2026                                   |
| ------------- | ------------------------------------------------- | -------------------------------------------------- | --------------------------------------------------- |
| `.default`    | Обычные запросы (самый частый случай)             | Кэширование, cookie, общие настройки системы       | Основной выбор для большинства [[API]]-запросов     |
| `.ephemeral`  | Запросы без хранения данных (приватный режим)     | Нет кэша, нет cookie, нет сохранённых credentials  | Режим инкогнито, разовые токены, тесты              |
| `.background` | Фоновые загрузки/выгрузки (файлы, большие данные) | Работает после выхода приложения, поддержка resume | Загрузка обновлений, резервные копии, большие файлы |

### Самые полезные свойства URLSessionConfiguration (2026)

| Свойство                          | Тип / Значение по умолчанию                      | Когда менять / пример значения                             | Влияние                                            |
| --------------------------------- | ------------------------------------------------ | ---------------------------------------------------------- | -------------------------------------------------- |
| `timeoutIntervalForRequest`       | TimeInterval (60 сек)                            | 15–30 сек для быстрых API, 300 сек для больших файлов      | Максимальное время на один запрос                  |
| `timeoutIntervalForResource`      | [[TimeInterval]] (7 дней)                        | 60–120 сек для большинства случаев                         | Общее время на всю операцию (включая редиректы)    |
| `httpMaximumConnectionsPerHost`   | [[Int]] (4–6)                                    | 1–2 для экономии батареи, 8–16 для высокой нагрузки        | Ограничение параллельных соединений к одному хосту |
| `allowsCellularAccess`            | [[Bool]] (true)                                  | false — только Wi-Fi (экономия трафика)                    | Запрет работы по мобильному интернету              |
| `httpAdditionalHeaders`           | [AnyHashable: Any]?                              | ["Authorization": "Bearer token"]                          | Заголовки по умолчанию для всех запросов           |
| `requestCachePolicy`              | URLRequest.CachePolicy (.useProtocolCachePolicy) | `.reloadIgnoringLocalCacheData` для свежих данных          | Управление кэшем                                   |
| `urlCache`                        | URLCache?                                        | URLCache(memoryCapacity: 0, diskCapacity: 0) для ephemeral | Отключение кэша                                    |
| `sessionSendsLaunchEvents`        | Bool (false)                                     | true — для background-сессий                               | Разрешает запуск приложения по фоновой задаче      |
| `discretionary`                   | Bool (false)                                     | true — низкоприоритетные фоновые задачи                    | Экономия батареи для background                    |
| `shouldUseExtendedBackgroundIdle` | Bool (false)                                     | true — для очень долгих background-загрузок                | iOS 13+                                            |

### Самые популярные и рекомендуемые конфигурации в 2026

#### 1. Стандартная конфигурация (самый частый случай)

```swift
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest = 30
config.timeoutIntervalForResource = 120
config.httpAdditionalHeaders = ["User-Agent": "MyApp/1.0"]
config.requestCachePolicy = .reloadIgnoringLocalCacheData

let session = URLSession(configuration: config)
```

#### 2. Фоновая загрузка (background session)

```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.example.background")
config.sessionSendsLaunchEvents = true
config.isDiscretionary = true
config.allowsCellularAccess = false

let backgroundSession = URLSession(configuration: config, delegate: self, delegateQueue: nil)
```

#### 3. Ephemeral (без кэша и cookie)

```swift
let config = URLSessionConfiguration.ephemeral
config.timeoutIntervalForRequest = 15
config.waitsForConnectivity = true

let privateSession = URLSession(configuration: config)
```

### Лучшие практики URLSessionConfiguration в Swift 2026

- **Не используй shared-сессию** (`URLSession.shared`) в продакшене — создавай свою с конфигурацией  
- **timeoutIntervalForRequest** — 15–60 сек (30 — золотая середина)  
- **timeoutIntervalForResource** — 60–300 сек (120 — часто оптимально)  
- **httpAdditionalHeaders** — добавляй User-Agent, Accept-Language, Authorization  
- **background** — используй уникальный identifier (bundle ID + суффикс)  
- **Swift 6 strict concurrency** — `URLSession` thread-safe, но [[Delegate]] должен быть [[Sendable]] или [[@MainActor]]  
- **[[Delegate]]** — используй `URLSessionTaskDelegate` для прогресса и аутентификации  
- **Документируйте** — пиши комментарий «URLSessionConfiguration с таймаутом 30 сек и без кэша»

**Короткий девиз 2026**:
> «URLSessionConfiguration — это когда ты хочешь сказать: «эта сессия должна работать именно так: с таким таймаутом, кэшем, заголовками и фоновым поведением».  
> В 2026 году почти никогда не используй URLSession.shared — всегда создавай свою конфигурацию.»
