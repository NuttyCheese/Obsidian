**Factory Method** (Фабричный метод) — это **порождающий паттерн проектирования** из классической книги GoF (Gang of Four), который определяет **интерфейс** для создания объекта, но **оставляет решение о том, какой именно класс инстанцировать**, на усмотрение подклассов.

В Swift 2026 году это один из самых живых и часто используемых паттернов, особенно в iOS/macOS-разработке, где нужно:

- создавать объекты разных типов в зависимости от контекста  
- скрывать детали создания (например, выбор между реальным API и моками)  
- соблюдать **OCP** (Open-Closed Principle) — добавлять новые типы без изменения существующего кода

### Классическая структура паттерна (GoF)

| Компонент             | Роль простыми словами                              | Swift-реализация 2026 года |
|-----------------------|-----------------------------------------------------|-----------------------------|
| **Product**           | Интерфейс / протокол создаваемого объекта           | protocol Product            |
| **ConcreteProduct**   | Конкретные реализации продукта                      | struct / class / actor      |
| **Creator**           | Абстрактный класс / протокол с фабричным методом    | protocol Creator { func factoryMethod() -> Product } |
| **ConcreteCreator**   | Конкретный создатель, который решает, какой продукт создать | class ConcreteCreator: Creator |

### Самые популярные реализации Factory Method в Swift 2026

#### Вариант 1 — Классический Factory Method (часто используется в legacy и UIKit)

```swift
protocol ViewControllerFactory {
    func makeViewController() -> UIViewController
}

class ProfileViewControllerFactory: ViewControllerFactory {
    func makeViewController() -> UIViewController {
        let vm = ProfileViewModel(repository: APIRepository())
        return ProfileViewController(viewModel: vm)
    }
}

class SettingsViewControllerFactory: ViewControllerFactory {
    func makeViewController() -> UIViewController {
        let vm = SettingsViewModel()
        return SettingsViewController(viewModel: vm)
    }
}

// Использование (например, в Coordinator)
class AppCoordinator {
    private let factory: ViewControllerFactory
    
    init(factory: ViewControllerFactory) {
        self.factory = factory
    }
    
    func showInitialScreen() {
        let vc = factory.makeViewController()
        // present / push vc
    }
}
```

#### Вариант 2 — Самый популярный в 2026 — **static func make** / **init?** (идиоматичный Swift)

```swift
protocol AnalyticsTracker {
    func track(_ event: String, parameters: [String: Any]?)
}

enum AnalyticsTrackerFactory {
    static func make(isDebug: Bool = false) -> any AnalyticsTracker {
        if isDebug {
            return DebugAnalyticsTracker()
        } else {
            return AmplitudeAnalyticsTracker(apiKey: "YOUR_KEY")
        }
    }
}

// Использование
let tracker = AnalyticsTrackerFactory.make(isDebug: BuildConfiguration.isDebug)
tracker.track("app_open", parameters: ["version": Bundle.main.version])
```

#### Вариант 3 — Factory Method с generics и primary associated types (Swift 5.7+)

```swift
protocol RepositoryFactory {
    associatedtype Repository: RepositoryProtocol
    
    func makeRepository() -> Repository
}

struct ProductionRepositoryFactory: RepositoryFactory {
    typealias Repository = APIRepository
    
    func makeRepository() -> APIRepository {
        APIRepository(session: URLSession.shared)
    }
}

struct TestRepositoryFactory: RepositoryFactory {
    typealias Repository = MockRepository
    
    func makeRepository() -> MockRepository {
        MockRepository()
    }
}

// Использование
let factory: any RepositoryFactory = BuildConfiguration.isDebug 
    ? TestRepositoryFactory() 
    : ProductionRepositoryFactory()

let repo = factory.makeRepository()  // тип выводится автоматически
```

### Когда Factory Method действительно нужен в 2026 году

| Сценарий                                      | Почему Factory Method выигрывает | Альтернатива (если не нужен) |
|-----------------------------------------------|-----------------------------------|-------------------------------|
| Создание ViewController / ViewModel по условию | Легко подменять в тестах / debug | SwiftUI @ViewBuilder / @Environment |
| Выбор реализации сервиса (real / mock / stub) | Полная изоляция создания          | swift-dependencies / Resolver |
| Поддержка разных окружений (dev / staging / prod) | Один интерфейс — разные фабрики   | Environment / Configuration   |
| Динамическая загрузка модулей / плагинов     | Добавление новых типов без изменения кода | Swift Package Manager plugins |
| A/B-тестирование UI / бизнес-логики           | Разные фабрики для разных групп   | Feature Flags + фабрики       |

### Лучшие практики Factory Method в Swift 2026

- **static func make** — самый идиоматичный и читаемый способ  
- **protocol + associatedtype** — если нужен сильный тип  
- **enum как фабрика** — для простых случаев (enum AnalyticsTrackerFactory { static func make() })  
- **Dependency Injection** — фабрика часто используется вместе с DI-контейнером  
- **Swift 6 strict concurrency** — фабрики должны возвращать Sendable-типы или actor  
- **Тестирование** — моки фабрик через протокол — очень просто  
- **Не создавай фабрику для всего** — YAGNI: фабрика нужна только когда есть выбор реализации

**Короткий девиз 2026**:
> «Factory Method в 2026 году — это когда ты хочешь сказать: «я создам тебе нужный объект, но тебе не важно, как именно я это сделаю».  
> Самый популярный стиль — static func make() или enum-фабрика.  
> Классический GoF с отдельными ConcreteCreator-классами почти не используется — его заменили более лёгкие и выразительные конструкции.»

Удачи с типобезопасным и гибким созданием объектов в Swift! 🏭