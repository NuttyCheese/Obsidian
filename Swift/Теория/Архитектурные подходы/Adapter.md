**Адаптер (Adapter)** — это структурный паттерн проектирования, который позволяет объектам с **несовместимыми интерфейсами** работать вместе.

Его главная задача — **сделать так, чтобы классы с разными интерфейсами могли взаимодействовать**, не меняя исходный код этих классов.

### Когда использовать Adapter (реальные сценарии 2025–2026)

| Ситуация                                                     | Почему нужен Adapter                                     | Пример из реальной iOS-разработки                |
| ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------ |
| Подключаем старую/чужую библиотеку с неудобным [[API]]       | Хотим использовать знакомый/современный интерфейс        | Старый сетевой клиент → новый протокол APIClient |
| Интегрируем сторонний SDK с неудобным/устаревшим интерфейсом | Хотим единый фасад для всей команды                      | Analytics SDK, Payment SDK, Maps SDK             |
| Переходим на новую версию библиотеки с breaking changes      | Не хотим переписывать весь проект сразу                  | Alamofire 4 → Alamofire 5                        |
| Работаем с legacy-кодом, который нельзя трогать              | Нужно интегрировать в новую архитектуру                  | Старый [[Core Data]] стек → новый репозиторий    |
| Нужно унифицировать работу с разными источниками данных      | Один интерфейс → разные реализации (локально, сеть, кэш) | ImageLoader (локальный файл / [[URL]] / PHAsset) |
| Тестирование (mocking) внешних зависимостей                  | Легко подменять реальные сервисы на заглушки             | NetworkService → MockNetworkService              |

### Два вида Adapter-паттерна

1. **Object Adapter** (композиция) — **самый рекомендуемый** в [[Swift]] 2025–2026  
   Использует **композицию** (держит внутри объект адаптируемого класса)

2. **Class Adapter** (наследование) — почти не используется в Swift  
   Требует множественного наследования → в Swift невозможно

### Классическая структура (Object Adapter)

```
                ┌────────────────────┐
                │   Client Code      │
                │ (использует Target)│
                └──────────┬─────────┘
                           │
                     ┌─────┴─────┐
                     │   Target  │ ← интерфейс, который ожидает клиент
                     └─────┬─────┘
                           │
                 ┌─────────┴─────────┐
                 │     Adapter       │ ← содержит Adaptee и реализует Target
                 └─────────┬─────────┘
                           │
                     ┌─────┴─────┐
                     │  Adaptee  │ ← старый/чужой класс с неудобным интерфейсом
                     └───────────┘
```

### Реальный пример из [[iOS]]-разработки (2026)

**Задача**:  
Есть старый сервис аналитики `OldAnalyticsTracker` с неудобным API.  
Хотим использовать современный интерфейс `AnalyticsService`.

```swift
// Целевой интерфейс (то, что хочет использовать клиент)
protocol AnalyticsService {
    func trackEvent(_ name: String, parameters: [String: Any]?)
    func trackScreen(_ screenName: String)
    func identifyUser(id: String, traits: [String: Any]?)
}

// Старый неудобный класс (Adaptee)
class OldAnalyticsTracker {
    func logEvent(name: String) { ... }
    func logEvent(name: String, meta: NSDictionary) { ... }
    func setUserId(_ userId: String) { ... }
    func setUserProperties(_ props: [String: AnyObject]) { ... }
}

// Адаптер
class AnalyticsAdapter: AnalyticsService {
    
    private let oldTracker: OldAnalyticsTracker
    
    init(oldTracker: OldAnalyticsTracker) {
        self.oldTracker = oldTracker
    }
    
    func trackEvent(_ name: String, parameters: [String: Any]?) {
        if let params = parameters as? [String: AnyObject] {
            oldTracker.logEvent(name: name, meta: params as NSDictionary)
        } else {
            oldTracker.logEvent(name: name)
        }
    }
    
    func trackScreen(_ screenName: String) {
        trackEvent("screen_view", parameters: ["screen_name": screenName])
    }
    
    func identifyUser(id: String, traits: [String: Any]?) {
        oldTracker.setUserId(id)
        if let traits = traits as? [String: AnyObject] {
            oldTracker.setUserProperties(traits)
        }
    }
}

// Использование
let oldTracker = OldAnalyticsTracker()
let analytics: AnalyticsService = AnalyticsAdapter(oldTracker: oldTracker)

// Теперь везде используем чистый современный интерфейс
analytics.trackEvent("user_login", parameters: ["method": "biometric"])
analytics.trackScreen("Profile")
analytics.identifyUser(id: "user123", traits: ["plan": "premium"])
```

### Когда стоит создавать Adapter (рекомендации 2026)

Создавать адаптер стоит, если выполняется хотя бы 2–3 условия:

- интерфейс внешней зависимости **неудобен** / **устарел** / **не соответствует вашему стилю**
- вы хотите **скрыть детали реализации** от остальной части приложения
- планируется **замена** этой зависимости в будущем (например, переход с Amplitude на Mixpanel)
- нужно **унифицировать** работу с несколькими похожими сервисами
- хотите **упростить тестирование** (mock-адаптеры пишутся гораздо легче)

### Лучшие практики Adapter в Swift 2026

- **Держите адаптер тонким** — только преобразование интерфейсов, минимум бизнес-логики  
- **Используйте протоколы** (Target) — это позволяет легко подменять реализации  
- **Избегайте** адаптеров, которые **добавляют** много новой логики — это уже не адаптер, а сервис/репозиторий  
- **Для сетевых клиентов** — чаще всего адаптеры превращаются в **Repository** / **DataSource**  
- **Для [[SwiftUI]]** — адаптеры часто реализуются как `@ObservableObject` / `@StateObject`  
- **Документируйте** — пишите комментарий «AnalyticsAdapter — адаптер старого трекера аналитики под новый AnalyticsService-протокол»

**Короткий итог 2026**:
> Adapter — это **преобразователь интерфейсов**, который позволяет несовместимым классам работать вместе.  
> В 2026 году:  
> - самый популярный вид — **Object Adapter** (композиция)  
> - используется для: унификации SDK, интеграции legacy-кода, упрощения тестирования  
> - в Swift — чаще всего реализуется через **протокол + класс-адаптер**  
> - это **один из самых полезных** паттернов при работе с чужим/старым кодом и сторонними библиотеками
