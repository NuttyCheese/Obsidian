**Wrapper** (обёртка) в [[Swift]] — это очень общий и часто используемый термин. Он может означать разные вещи в зависимости от контекста. Вот самые популярные и актуальные на 2026 год варианты использования слова «wrapper» в [[iOS]]/Swift-разработке.

### 1. Wrapper как типобезопасная обёртка над протоколом (самый частый случай)

```swift
// Классический Any<Protocol> wrapper (type erasure)
struct AnyAnalyticsTracker: AnalyticsTracker {
    private let _trackEvent: (String, [String: Any]?) -> Void
    
    init<T: AnalyticsTracker>(_ base: T) {
        self._trackEvent = base.trackEvent
    }
    
    func trackEvent(_ name: String, parameters: [String: Any]?) {
        _trackEvent(name, parameters)
    }
}

// Использование
let tracker: any AnalyticsTracker = AnyAnalyticsTracker(AmplitudeTracker())
tracker.trackEvent("app_open", parameters: nil)
```

**Альтернативы 2026 года** (гораздо чаще используют):
- `some AnalyticsTracker` (opaque type)  
- [[@MainActor]] классы  
- [[actor]] для сервисов  
- **swift-dependencies** / **Factory** вместо ручных wrapper’ов

### 2. Property Wrapper (@Wrapper)

Самый популярный вид wrapper в 2026 году — **property wrappers** (`@Published`, `@State`, `@ObservedObject`, `@Environment`, `@Default`, `@UserDefault` и т.д.)

```swift
@propertyWrapper
struct Clamped<T: Comparable> {
    private var value: T
    private let range: ClosedRange<T>
    
    var wrappedValue: T {
        get { value }
        set { value = max(range.lowerBound, min(newValue, range.upperBound)) }
    }
    
    init(wrappedValue: T, range: ClosedRange<T>) {
        self.value = max(range.lowerBound, min(wrappedValue, range.upperBound))
        self.range = range
    }
}

struct Settings {
    @Clamped(range: 0...100) var volume: Int = 50
    @Clamped(range: 1...10) var brightness: Double = 5.0
}
```

### 3. Wrapper как UI-компонент ([[SwiftUI]])

```swift
struct LoadingWrapper<Content: View>: View {
    let isLoading: Bool
    let content: () -> Content
    
    var body: some View {
        ZStack {
            content()
            if isLoading {
                ProgressView()
                    .scaleEffect(1.5)
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
                    .background(.ultraThinMaterial)
            }
        }
    }
}

// Использование
LoadingWrapper(isLoading: viewModel.isLoading) {
    List(viewModel.items) { item in
        Text(item.title)
    }
}
```

### 4. Wrapper над сетевым слоем / [[API]]-клиентом

```swift
actor SecureNetworkWrapper: NetworkService {
    private let baseService: NetworkService
    private let authToken: String?
    
    init(base: NetworkService, token: String?) {
        self.baseService = base
        self.authToken = token
    }
    
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var request = endpoint.request
        if let token = authToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        return try await baseService.fetch(endpoint)
    }
}
```

### 5. Wrapper для тестирования / моков (Mock Wrapper)

```swift
struct MockAnalytics: AnalyticsService {
    var events: [(name: String, params: [String: Any]?)] = []
    
    func track(_ name: String, parameters: [String: Any]?) {
        events.append((name, parameters))
    }
}

// В тестах
let mock = MockAnalytics()
let service = AnalyticsWrapper(real: AmplitudeService(), mock: mock)
```

### Лучшие практики «wrapper» в Swift 2026

- **Предпочитай property wrapper (@Wrapper)** — для повторяющейся логики свойств  
- **Используй some вместо any** — когда wrapper возвращает протокол  
- **actor + wrapper** — для безопасного shared state  
- **[[final]] [[class]] + @unchecked Sendable** — для [[Singleton]]-wrapper’ов  
- **Swift 6 strict concurrency** — wrapper должен быть [[Sendable]] или изолирован (@MainActor / actor)  
- **Не злоупотребляй** — каждый wrapper добавляет сложность → используй только когда реально нужно скрыть детали  
- **Документируйте** — пиши комментарий «Wrapper — добавляет токен авторизации и логирование»

**Короткий девиз 2026**:
> «Wrapper — это когда ты берёшь существующий объект/сервис/логику и оборачиваешь его в новый слой: добавляешь кэш, логирование, авторизацию, моки, безопасность или удобный API.  
> В 2026 году самые популярные wrapper’ы — property wrappers (@Published, @Default, @Clamped), actor-wrapper’ы и some-возвращаемые функции.  
> Главное правило: wrapper должен делать код **проще**, а не сложнее.»
