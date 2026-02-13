### 1. Зачем Builder нужен в Swift в 2026 году

Builder решает две главные проблемы:

- **Слишком много параметров в [[init]]** → конструктор становится нечитаемым  
- **Хочется создавать объект разными способами** (с разными наборами полей, с дефолтными значениями, с валидацией, с цепочкой методов)

Самые частые сценарии в [[iOS]]-приложениях 2026 года, где Builder активно используется:

- сложные модели запросов ([[API]] Request, [[GraphQL]] variables)  
- конфигурация UI-компонентов ([[UIView]], [[SwiftUI]] View modifiers)  
- создание объектов с большим количеством опциональных полей (UserProfile, PaymentRequest, NotificationContent)  
- fluent-строители в библиотеках ([[Alamofire]], [[Kingfisher]], SwiftUI ViewBuilder, Composable Architecture)  
- тестирование (создание тестовых объектов с частично заполненными полями)

### 2. Классическая структура паттерна Builder (4 элемента)

| Компонент           | Роль в классическом паттерне                             | Актуальность в Swift 2026                          |
| ------------------- | -------------------------------------------------------- | -------------------------------------------------- |
| **Product**         | Итоговый сложный объект, который мы строим               | Обычно struct / class                              |
| **Builder**         | Протокол / абстрактный класс с методами-шагами           | Обычно протокол                                    |
| **ConcreteBuilder** | Конкретная реализация, которая знает, как строить объект | Обычно один или несколько классов / структур       |
| **Director**        | Класс, который знает порядок шагов строительства         | **Почти никогда не нужен** в современном [[Swift]] |

**Важно**: в современном Swift **Director** почти полностью исчез — его роль взяли на себя **chainable методы** и **Result Builder**.

### 3. Самые популярные и рекомендуемые реализации Builder в Swift 2026

#### Вариант 1 — Классический Builder с протоколом (редко, но встречается)

```swift
protocol RequestBuilder {
    func setMethod(_ method: String) -> Self
    func setPath(_ path: String) -> Self
    func setHeader(_ key: String, value: String) -> Self
    func setBody(_ body: Data?) -> Self
    func build() -> URLRequest
}

class URLRequestBuilder: RequestBuilder {
    private var request = URLRequest(url: URL(string: "https://api.com")!)
    
    func setMethod(_ method: String) -> Self {
        request.httpMethod = method
        return self
    }
    
    func setPath(_ path: String) -> Self {
        request.url = URL(string: "https://api.com" + path)
        return self
    }
    
    // ... остальные методы
    
    func build() -> URLRequest {
        return request
    }
}

// Использование
let request = URLRequestBuilder()
    .setMethod("POST")
    .setPath("/users")
    .setHeader("Authorization", value: "Bearer token")
    .build()
```

**Минусы в 2026**: слишком много boilerplate, не idiomatic для Swift.

#### Вариант 2 — Самый популярный и рекомендуемый в 2026 — **Fluent Builder** (цепочка методов)

```swift
struct UserProfile {
    let id: UUID
    let name: String
    let email: String?
    let avatarURL: URL?
    let isPremium: Bool
    let birthDate: Date?
    // много других полей...
    
    // приватный init — снаружи только через builder
    private init(id: UUID, name: String, email: String?, avatarURL: URL?, isPremium: Bool, birthDate: Date?) {
        self.id = id
        self.name = name
        self.email = email
        self.avatarURL = avatarURL
        self.isPremium = isPremium
        self.birthDate = birthDate
    }
    
    // MARK: - Fluent Builder
    
    static func builder(id: UUID, name: String) -> Builder {
        Builder(id: id, name: name)
    }
    
    final class Builder {
        private var id: UUID
        private var name: String
        private var email: String?
        private var avatarURL: URL?
        private var isPremium = false
        private var birthDate: Date?
        
        fileprivate init(id: UUID, name: String) {
            self.id = id
            self.name = name
        }
        
        func email(_ email: String) -> Self {
            self.email = email
            return self
        }
        
        func avatarURL(_ url: URL) -> Self {
            self.avatarURL = url
            return self
        }
        
        func premium(_ isPremium: Bool = true) -> Self {
            self.isPremium = isPremium
            return self
        }
        
        func birthDate(_ date: Date) -> Self {
            self.birthDate = date
            return self
        }
        
        func build() -> UserProfile {
            UserProfile(
                id: id,
                name: name,
                email: email,
                avatarURL: avatarURL,
                isPremium: isPremium,
                birthDate: birthDate
            )
        }
    }
}

// Использование — читаемо и красиво
let profile = UserProfile.builder(id: UUID(), name: "Alex")
    .email("alex@example.com")
    .avatarURL(URL(string: "https://...")!)
    .premium()
    .birthDate(Date(timeIntervalSince1970: 800_000_000))
    .build()
```

#### Вариант 3 — Самый современный и idiomatic в 2026 — **Result Builder** (как в SwiftUI)

```swift
@resultBuilder
enum ProfileBuilder {
    static func buildBlock(_ components: ProfileComponent...) -> [ProfileComponent] {
        components
    }
}

protocol ProfileComponent {
    func apply(to profile: inout UserProfile)
}

struct EmailComponent: ProfileComponent {
    let email: String
    func apply(to profile: inout UserProfile) {
        profile.email = email
    }
}

struct PremiumComponent: ProfileComponent {
    func apply(to profile: inout UserProfile) {
        profile.isPremium = true
    }
}

// и т.д.

@ProfileBuilder
func buildProfile(id: UUID, name: String, @ProfileBuilder _ builder: () -> [ProfileComponent]) -> UserProfile {
    var profile = UserProfile(id: id, name: name, email: nil, avatarURL: nil, isPremium: false, birthDate: nil)
    
    for component in builder() {
        component.apply(to: &profile)
    }
    
    return profile
}

// Использование — почти как SwiftUI
let profile = buildProfile(id: UUID(), name: "Alex") {
    EmailComponent(email: "alex@example.com")
    PremiumComponent()
    // можно добавлять новые компоненты без изменения кода
}
```

### 4. Сравнение подходов к Builder в Swift 2026

| Подход                                     | Читаемость | Кол-во boilerplate | Тестируемость | Поддержка Swift 6+ | Рекомендация 2026    |
| ------------------------------------------ | ---------- | ------------------ | ------------- | ------------------ | -------------------- |
| Классический Builder (протокол + Director) | ★★★☆☆      | ★★★★☆              | ★★★★☆         | ★★★★☆              | Редко                |
| Fluent Builder (цепочка методов)           | ★★★★★      | ★★☆☆☆              | ★★★★★         | ★★★★★              | Основной выбор       |
| Result Builder (как в SwiftUI)             | ★★★★★      | ★☆☆☆☆              | ★★★★☆         | ★★★★★              | Для DSL-подобных API |
| [[Copy-on-write]] + mutating методы        | ★★★★☆      | ★☆☆☆☆              | ★★★★☆         | ★★★★★              | Простые модели       |

### 5. Лучшие практики Builder в Swift 2026

- **Fluent Builder** — самый универсальный и читаемый стиль  
- **Result Builder** — если нужен **DSL-подобный** синтаксис (как в SwiftUI, [[TCA]], Kingfisher, Alamofire)  
- **Скрытый init** — делайте основной init приватным, чтобы заставлять использовать builder  
- **@MainActor / actor** — если модель используется в UI или многопотоке — делайте builder тоже [[@MainActor]]  
- **Тестирование** — builder идеально подходит для создания тестовых объектов с частичным заполнением  
- **Документируйте** — пишите в документации модели «рекомендуется создавать через builder»  
- **Swift 6 strict concurrency** — builder должен быть Sendable или использовать actor, если содержит состояние  

**Короткий девиз 2026**:
> «Builder в 2026 году — это когда ты хочешь сказать: «создавай меня красиво, безопасно и без боли в конструкторе».  
> Самый популярный стиль — fluent chainable методы + приватный init.  
> Для DSL-подобных API — Result Builder.»
