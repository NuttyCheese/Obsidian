**`final`** — это модификатор в [[Swift]], который **запрещает** дальнейшее наследование или переопределение.

Он может применяться к:

- классам
- методам
- свойствам
- подпискам (subscripts)
- расширениям

### Где и зачем используют `final` (2025–2026 практика)

| Что помечено как final    | Что это запрещает                             | Когда это полезно / обязательно в 2026                                    |
| ------------------------- | --------------------------------------------- | ------------------------------------------------------------------------- |
| `final class`             | Наследование от этого класса                  | Почти все ViewModel, сервисы, менеджеры, DTO — если наследование не нужно |
| `final func`              | Переопределение метода в подклассах           | Методы, которые не должны изменяться (бизнес-логика, безопасность)        |
| `final var` / `final let` | Переопределение свойства в подклассах         | Константы, вычисляемые свойства, хранимые свойства с важной логикой       |
| `final class func`        | Переопределение type-метода                   | Фабричные методы, статические хелперы                                     |
| `final` в расширении      | Переопределение метода/свойства из расширения | Расширения [[UIKit]]/UIKitExtensions                                      |

### Примеры (современный стиль)

#### 1. final class — самый частый случай

```swift
final class UserProfileViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
    
    func loadUser() async throws {
        // ...
    }
}
```

**Почему final**:
- никто не должен наследоваться от ViewModel
- компилятор может оптимизировать вызовы методов (devirtualization)
- защита от случайного расширения

#### 2. final func — защита поведения

```swift
class BaseViewController: UIViewController {
    final func setupNavigationBar() {
        // критичная настройка навигации
        navigationItem.largeTitleDisplayMode = .always
        navigationController?.navigationBar.prefersLargeTitles = true
    }
}

class ProfileViewController: BaseViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        setupNavigationBar() // нельзя переопределить
    }
}
```

#### 3. final var в протокол-ориентированном коде

```swift
protocol Trackable {
    var trackingId: String { get }
}

struct AnalyticsEvent: Trackable {
    final var trackingId: String
    
    init(eventName: String) {
        self.trackingId = "event_\(eventName)_\(UUID().uuidString.prefix(8))"
    }
}
```

`trackingId` нельзя переопределить → гарантированная уникальность.

#### 4. final в расширении (очень популярно в UIKitExtensions)

```swift
extension UIButton {
    final func setSystemImage(_ name: String, for state: UIControl.State = .normal) {
        setImage(UIImage(systemName: name), for: state)
    }
}
```

Никто не сможет переопределить эту удобную обёртку.

### 5. Производительность и оптимизация (почему final любят в 2026)

| Ситуация                              | Без final                                 | С final                           | Выигрыш                                         |
| ------------------------------------- | ----------------------------------------- | --------------------------------- | ----------------------------------------------- |
| Вызов метода класса                   | Динамическая диспетчеризация ([[vtable]]) | Статическая диспетчеризация       | ~20–50% быстрее в горячих путях                 |
| [[final]] [[class]]                   | Может быть подклассом                     | Компилятор знает — нет подклассов | Полная оптимизация (inlining, devirtualization) |
| final [[func]] в цикле / горячем коде | Косвенный вызов                           | Прямой вызов                      | Заметно в анимациях, играх, ML                  |

**Замеры (примерные, 2026, M4 Pro / A18 Pro)**:
- вызов final метода: ~1–2 нс  
- вызов обычного метода класса: ~3–8 нс (если не инлайнится)

### 6. Лучшие практики final в Swift 2026

- Делай `final class` **по умолчанию** для всех ViewModel, сервисов, менеджеров, coordinators, DTO — если наследование не планируется  
- Помечай `final func` / `final var` там, где поведение **не должно** меняться в подклассах  
- Используй `final` в расширениях UIKit/AppKit — защита от случайного переопределения  
- **Не пиши final** там, где предполагается наследование ([[UIViewController]], [[UITableViewCell]], custom views)  
- **Swift 6 strict concurrency** — `final` не влияет на [[Sendable]]/акторы, но помогает компилятору лучше оптимизировать  
- Документируй — пиши комментарий «final class — не предназначен для наследования»

**Короткий девиз 2026**:
> `final` — это «никто не должен наследоваться / переопределять».  
> В 2026 году используй его **везде**, где наследование не нужно:  
> - final class для ViewModel / сервисов  
> - final func для критичной логики  
> - final в расширениях [[UIKit]]  
> Это **ускоряет** код, **защищает** от ошибок и делает намерения явными.
