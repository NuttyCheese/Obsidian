#pattern #architectural_approaches 
Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по **SOLID** в Swift — с примерами, схемами, реальными iOS-сценариями и рекомендациями для Swift 6+.

### 1. Что такое SOLID и почему он актуален в 2026 году

**SOLID** — это акроним пяти принципов объектно-ориентированного дизайна, предложенных Робертом Мартином (Uncle Bob) в начале 2000-х:

- **S** — Single Responsibility Principle  
- **O** — Open-Closed Principle  
- **L** — Liskov Substitution Principle  
- **I** — Interface Segregation Principle  
- **D** — Dependency Inversion Principle  

В 2026 году SOLID **не устарел** — наоборот, он стал **ещё важнее** благодаря:

- **Swift 6 strict concurrency** — без SOLID легко получить data race  
- **TCA / Composable Architecture** — требует строгого соблюдения SOLID  
- **Clean Swift / VIPER / MVVM-C** — SOLID лежит в основе  
- **Долгоживущие проекты** (5–10+ лет) — без SOLID код становится неподдерживаемым  
- **Команды > 5 человек** — SOLID минимизирует конфликты merge и регрессии

### 2. Краткое сравнение принципов SOLID в Swift 2026

| Принцип | Полное название | Краткая суть | Самая частая ошибка | Лучшая практика в Swift 2026 |
|--------|------------------|--------------|----------------------|-------------------------------|
| **S**  | Single Responsibility | Один класс — одна ответственность | God Class / Massive VC | Маленькие классы + протоколы |
| **O**  | Open-Closed      | Открыт для расширения, закрыт для модификации | if-else по типу / switch по строке | Протоколы + композиция |
| **L**  | Liskov Substitution | Подтипы взаимозаменяемы с базовым типом | fatalError в подклассе / сужение предусловий | Узкие протоколы + композиция |
| **I**  | Interface Segregation | Не заставляй реализовывать ненужные методы | «Fat Protocol» на 20 методов | 5 протоколов по 3 метода лучше 1 на 15 |
| **D**  | Dependency Inversion | Зависеть от абстракций, а не от деталей | `let api = URLSession.shared` внутри VM | Dependency Injection через протоколы |

### 3. Подробное объяснение каждого принципа с примерами

#### S — Single Responsibility Principle

**Класс должен иметь только одну причину для изменения.**

```swift
// Нарушение SRP — один класс делает всё
class ProfileManager {
    func fetchUser() async throws -> User { ... }     // сеть
    func validateUser(_ user: User) -> Bool { ... }   // валидация
    func saveToCoreData(_ user: User) { ... }         // БД
    func formatName(_ name: String) -> String { ... } // форматирование
    func updateUI(with user: User) { ... }            // UI
}

// Правильно — каждый класс делает одну вещь
actor UserRepository { func save(_ user: User) async throws { ... } }
struct UserValidator { func validate(_ user: User) -> Bool { ... } }
struct UserFormatter { func formatName(_ name: String) -> String { ... } }
@MainActor class ProfileViewModel { 
    func update(with user: User) { ... } 
}
```

#### O — Open-Closed Principle

**Открыт для расширения, закрыт для модификации.**

```swift
// Нарушение — добавление новой оплаты ломает класс
class PaymentProcessor {
    func process(_ amount: Double, method: String) {
        switch method {
        case "card":    print("Карта")
        case "apple":   print("Apple Pay")
        // case "crypto": ← пришлось бы менять класс
        }
    }
}

// Правильно — расширение через новый тип
protocol PaymentMethod { func process(_ amount: Double) }
struct CardPayment: PaymentMethod { func process(_ amount: Double) { ... } }
struct CryptoPayment: PaymentMethod { func process(_ amount: Double) { ... } } // новый тип — старый код не трогаем

class PaymentProcessor {
    let method: any PaymentMethod
    init(method: any PaymentMethod) { self.method = method }
    func pay(_ amount: Double) { method.process(amount) }
}
```

#### L — Liskov Substitution Principle

**Подтип можно подставить вместо базового без изменения поведения.**

```swift
// Нарушение — подстановка ломается
protocol Bird { func fly() }
class Ostrich: Bird { func fly() { fatalError("Не летаю") } } // ← нельзя подставить

// Правильно — страус не реализует Flyable
protocol Flyable { func fly() }
class Sparrow: Flyable { func fly() { ... } }
class Ostrich {} // не реализует Flyable — LSP соблюдается
```

#### I — Interface Segregation Principle

**Не заставляй реализовывать ненужные методы.**

```swift
// Нарушение — «жирный» протокол
protocol Worker {
    func work()
    func eat()
    func sleep()
    func rechargeBattery()
}

// Правильно — узкие протоколы
protocol Workable { func work() }
protocol Rechargeable { func rechargeBattery() }

class Human: Workable, Rechargeable { ... }     // только нужные
class Robot: Workable, Rechargeable { ... }     // только нужные
```

#### D — Dependency Inversion Principle

**Зависеть от абстракций, а не от деталей.**

```swift
// Нарушение — жёсткая зависимость
class ViewModel {
    private let api = URLSession.shared // конкретная реализация
}

// Правильно — зависимость через протокол
protocol NetworkService { func fetch() async throws -> Data }
class APIService: NetworkService { ... }

class ViewModel {
    private let service: any NetworkService
    init(service: any NetworkService) { self.service = service }
}
```

### 4. Визуальная схема SOLID (2026 стиль)

```mermaid
flowchart TD
    SOLID[SOLID Principles] --> S["S — Single Responsibility<br>Один класс — одна ответственность"]
    SOLID --> O["O — Open-Closed<br>Открыт для расширения, закрыт для модификации"]
    SOLID --> L["L — Liskov Substitution<br>Подтипы взаимозаменяемы"]
    SOLID --> I["I — Interface Segregation<br>Маленькие, узкие протоколы"]
    SOLID --> D["D — Dependency Inversion<br>Зависеть от абстракций"]

    S --> "Маленькие классы + протоколы"
    O --> "Протоколы + композиция"
    L --> "Узкие протоколы + композиция"
    I --> "Протоколы по 2–5 методов"
    D --> "Dependency Injection через протоколы"
```

### 5. Лучшие практики SOLID в Swift 2026

- **S** — классы/актёры < 200 строк, одна причина изменения  
- **O** — точка расширения = протокол + новая реализация  
- **L** — подтип можно подставить вместо базового без изменения поведения  
- **I** — протоколы маленькие (2–5 методов), названия конкретные  
- **D** — все зависимости через протоколы и constructor injection  
- **Swift 6 strict concurrency** — SOLID + actor + маленькие классы = почти 100% отсутствие data race  
- **Не бойтесь** создавать 50–100 маленьких классов/протоколов — Xcode справится  
- **Тестирование** — маленькие классы = маленькие, быстрые, независимые тесты  
- **Документируйте** — пишите в документации класса/протокола «отвечает только за X»

**Короткий девиз 2026**:
> «SOLID в 2026 году — это когда ты пишешь код так, чтобы через 5 лет его мог поддерживать новый разработчик без боли.  
> Без SOLID в Swift 6+ писать долгоживущее, тестируемое и масштабируемое приложение уже считается плохим тоном.»

Удачи с чистой, современной и поддерживаемой архитектурой в Swift! 🏛️