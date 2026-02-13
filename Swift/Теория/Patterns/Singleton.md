### 1. Классическая реализация Singleton в [[Swift]] (та самая, которую все знают)

```swift
final class MySingleton {
    static let shared = MySingleton()
    
    private init() {
        // приватный инициализатор
        // здесь можно добавить логику инициализации
    }
    
    func doSomething() {
        print("Работаю как синглтон")
    }
}
```

**Ключевые особенности этой реализации** (и почему она потокобезопасна):

- `static let shared` — инициализируется **лениво** (при первом обращении)  
- В Swift **статическая инициализация** потокобезопасна **с версии Swift 1.0** ([[GCD]] dispatch_once под капотом до Swift 5.1, затем встроенный механизм)  
- `private init()` — предотвращает создание экземпляров извне  
- `final class` — запрещает наследование (очень важно для синглтонов)

### 2. Почему классический Singleton в 2026 году считается антипаттерном

| Проблема                                                                           | Последствия в 2026 году                            | Почему это критично                   |
| ---------------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------- |
| Глобальное состояние                                                               | Очень сложно тестировать, мокать, подменять        | Unit-тесты становятся интеграционными |
| Нарушение принципа инверсии зависимостей ([[Dependency Inversion Principle\|DIP]]) | ViewModel / UseCase зависят от глобального объекта | Нарушение SOLID                       |
| Проблемы с многопоточностью и actor                                                | Доступ к shared из разных actor → [[data race]]    | Swift 6 strict concurrency ловит это  |
| Невозможность scoped / per-feature singletons                                      | Один синглтон на всё приложение                    | Невозможны A/B-тесты, разные конфиги  |
| Сложно контролировать жизненный цикл                                               | Нет [[deinit]], нет явного создания/уничтожения    | Утечки памяти, [[retain cycle]]       |
| Плохая читаемость и предсказуемость                                                | `MySingleton.shared.doSomething()` в любом месте   | Код становится «спагетти»             |

**Вывод 2026 года от большинства senior iOS-разработчиков и Apple**:
> «Если тебе нужен Singleton — скорее всего, тебе нужен **actor** с `static let shared` или **Dependency Injection** через протокол.»  
> Классический `static let shared` с `private init()` теперь считается **legacy-антипаттерном** в новом коде.

### 3. Современные альтернативы Singleton в Swift 2026

#### Альтернатива №1 — Самая рекомендуемая: **actor + static shared**

```swift
actor AnalyticsService {
    static let shared = AnalyticsService()
    
    private var events: [String] = []
    
    func track(_ event: String) {
        events.append(event)
        print("Tracked:", event)
    }
    
    func flush() {
        // отправка на сервер
        events.removeAll()
    }
}

// Использование (безопасно из любого контекста)
Task {
    await AnalyticsService.shared.track("screen_view")
}
```

**Почему это лучше классического Singleton**:

- **потокобезопасно** по умолчанию (всё состояние изолировано)  
- **Sendable** автоматически  
- можно добавить `deinit` (если очень нужно)  
- можно тестировать через протокол (см. ниже)

#### Альтернатива №2 — Dependency Injection через протокол (идеально для тестов)

```swift
protocol AnalyticsTracking {
    func track(_ event: String)
    func flush()
}

actor AnalyticsService: AnalyticsTracking {
    static let shared = AnalyticsService()
    
    private var events: [String] = []
    
    func track(_ event: String) {
        events.append(event)
    }
    
    func flush() {
        events.removeAll()
    }
}

// Внедрение через DI
@MainActor
class AnalyticsManager {
    private let tracker: any AnalyticsTracking
    
    init(tracker: any AnalyticsTracking = AnalyticsService.shared) {
        self.tracker = tracker
    }
    
    func trackScreen(_ name: String) {
        tracker.track("screen_view: \(name)")
    }
}

// Тест
let mock = MockAnalyticsTracker()
let manager = AnalyticsManager(tracker: mock)
```

#### Альтернатива №3 — `@globalActor` (если нужен глобальный синглтон с изоляцией)

```swift
@globalActor
actor AnalyticsActor {
    static let shared = AnalyticsActor()
    
    private var events: [String] = []
    
    func track(_ event: String) {
        events.append(event)
    }
}

@AnalyticsActor
func trackGlobal(_ event: String) {
    // автоматически на AnalyticsActor.shared
    await AnalyticsActor.shared.track(event)
}
```

### 4. Когда классический Singleton всё ещё оправдан в 2026

| Сценарий                                                 | Почему всё ещё можно использовать | Рекомендация                  |
| -------------------------------------------------------- | --------------------------------- | ----------------------------- |
| Поддержка legacy-кода (iOS 12–14)                        | Совместимость                     | Не трогать                    |
| Внутренние утилиты без состояния (Logger, DateFormatter) | Очень низкий риск                 | Можно, но лучше extension     |
| Третьи библиотеки / SDK                                  | Их [[API]] требует shared         | Обернуть в [[actor]] / DI     |
| Очень простые случаи (Analytics без сложной логики)      | Минимум кода                      | actor.shared предпочтительнее |


### 5. Лучшие практики Singleton в Swift 2026

- **actor + static let shared** — основной и рекомендуемый способ  
- **Dependency Injection** — всегда, когда нужна тестируемость  
- **Никогда** не делайте `static var shared` с `var` — это глобальное изменяемое состояние  
- **@globalActor** — если нужен глобальный контекст с изоляцией  
- **Тестирование** — всегда мокайте через протоколы, а не трогайте `.shared`  
- **Swift 6 strict concurrency** — классический Singleton с `var` почти всегда вызовет предупреждения  
- **Документируйте** — пишите комментарий «shared — глобальный синглтон, используйте с осторожностью»

**Короткий девиз 2026**:
> «Singleton в 2026 году — это когда тебе нужен один экземпляр на всё приложение.  
> Лучший способ — actor + static let shared.  
> Классический static let shared с private init — legacy, используй только в поддержке старого кода.»
