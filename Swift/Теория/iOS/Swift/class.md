**`class`** в Swift — это **ссылочный тип** ([[reference type]]), который используется для создания объектов с **общим состоянием** и **поведением**.  
Классы — основа объектно-ориентированного программирования в [[Swift]] и [[UIKit]]/[[AppKit]].

### 1. Ключевые отличия class от struct / enum (таблица 2026)

| Характеристика     | class (Reference Type)                                       | [[struct]] / [[enum]] ([[Value Type]])                   | Когда выбрать class в 2026                        |
| ------------------ | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------- |
| Семантика передачи | По ссылке (один объект в памяти)                             | По значению (копируется)                                 | Нужно общее состояние                             |
| Наследование       | Полное (single inheritance)                                  | Нет наследования                                         | Иерархия типов ([[UIView]], [[UIViewController]]) |
| [[Deinit]]         | Есть (`deinit`) — можно освобождать ресурсы                  | Нет                                                      | Работа с файлами, сетью, наблюдателями            |
| Identity           | Сравнивается по ссылке (`===`, `!==`)                        | Сравнивается по значению (`==`, `!=`)                    | Нужно знать, один ли объект                       |
| Mutability         | Можно менять свойства даже через `let` ссылку                | `let` объект полностью неизменяем                        | Объекты с внутренним состоянием                   |
| Thread safety      | Требует ручной синхронизации / actor                         | Безопасен по умолчанию (если нет мутабельных свойств)    | UIKit — почти всегда class                        |
| [[ARC]] overhead   | Каждый объект — отдельная аллокация + [[retain]]/[[release]] | Копируется только при изменении ([[Copy-On-Write\|COW]]) | Много маленьких объектов — лучше struct           |

### 2. Когда использовать class в 2026 году (рекомендации Apple)

| Сценарий                                  | Почему именно class                                             | Примеры из реальных приложений                      |
| ----------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------- |
| UIKit / AppKit элементы интерфейса        | Все контроллеры, вью, жесты, layers — классы                    | `UIView`, `UIViewController`, `UIButton`, `CALayer` |
| Модели с общим состоянием                 | Несколько частей приложения работают с одним объектом           | `UserSession`, `CartManager`, `AudioPlayer`         |
| Наследование и полиморфизм                | Нужна иерархия классов с переопределением методов               | `BaseViewController` → `ProfileVC`, `SettingsVC`    |
| Работа с ресурсами (файлы, сеть, камера)  | Нужно `deinit` для освобождения (закрыть файл, отменить запрос) | `AVPlayer`, `URLSessionTask`, `PHAsset`             |
| KVO, [[NotificationCenter]], [[delegate]] | Требуется стабильная ссылка (identity)                          | `observe(\.value)`, `NotificationCenter` observers  |
| Долгоживущие объекты приложения           | Singleton’ы, менеджеры, сервисы                                 | [[AppDelegate]], [[SceneDelegate]], `Analytics`     |

### 3. Самый современный и рекомендуемый паттерн класса 2026 года

```swift
@MainActor
final class UserProfileViewModel {
    
    // MARK: - Properties
    private(set) var user: User?
    private(set) var isLoading = false
    private(set) var errorMessage: String?
    
    // MARK: - Published / Observable (для SwiftUI / Combine)
    @Published var avatarURL: URL?
    @Published var displayName: String = "Гость"
    
    // MARK: - Dependencies (внедрение зависимостей)
    private let authService: AuthService
    private let userService: UserService
    
    // MARK: - Initialization
    init(authService: AuthService, userService: UserService) {
        self.authService = authService
        self.userService = userService
    }
    
    // MARK: - Public Methods
    func loadProfile() async {
        guard !isLoading else { return }
        isLoading = true
        errorMessage = nil
        
        do {
            let currentUser = try await authService.currentUser()
            let profile = try await userService.fetchProfile(for: currentUser.id)
            
            await MainActor.run {
                self.user = profile
                self.displayName = profile.name
                self.avatarURL = profile.avatarURL
            }
        } catch {
            await MainActor.run {
                self.errorMessage = error.localizedDescription
            }
        }
        
        await MainActor.run {
            self.isLoading = false
        }
    }
    
    // MARK: - Deinitializer (редко, но полезно)
    deinit {
        print("UserProfileViewModel deallocated")
        // Отмена всех задач, отписка от уведомлений и т.д.
    }
}
```

### 4. Полный список самых важных возможностей class

| Возможность                                     | Синтаксис / Пример                       | Когда использовать в 2026                           |
| ----------------------------------------------- | ---------------------------------------- | --------------------------------------------------- |
| Наследование                                    | `class Child: Parent { ... }`            | UIKit/AppKit, ViewModel’ы с общей логикой           |
| Переопределение методов                         | `override func viewDidLoad() { ... }`    | Кастомизация поведения родителя                     |
| [[final]] (запрет наследования/переопределения) | `final class`, `final func`, `final var` | Оптимизация + защита от нежелательного наследования |
| required init                                   | `required init(coder: NSCoder)`          | Обязательные инициализаторы (NSCoding, Storyboard)  |
| convenience init                                | `convenience init(name: String)`         | Удобные конструкторы                                |
| deinit                                          | `deinit { cleanup() }`                   | Освобождение ресурсов (KVO, observers, files)       |
| Сравнение по идентичности                       | `object1 === object2`                    | Проверка, один ли объект                            |
| [[Weak]] / [[unowned]] ссылки                   | `weak var delegate: Delegate?`           | Предотвращение [[retain cycle]]                     |
| [[Lazy]] свойства                               | `lazy var expensiveObject = Expensive()` | Отложенная инициализация                            |

### 5. Лучшие практики class в Swift 2026

- **Делай классы final по умолчанию** — если наследование не нужно (улучшает производительность и безопасность)  
- **Используй @MainActor для UI-классов** — это требование Swift 6 strict concurrency  
- **Предпочитай struct для моделей данных** — class только если нужен общий mutable state или наследование  
- **Не делай слишком большие классы** — разбивай на маленькие с одной ответственностью (SOLID)  
- **Используй dependency injection** — передавай сервисы через init, а не глобальные singletons  
- **Документируйте deinit** — пиши `print` или логирование в deinit для отладки утечек  
- **Swift 6 strict concurrency** — class должен быть `Sendable` или помечен `@MainActor` / `actor`  
- **Документируйте** — пиши комментарий «final class UserProfileViewModel — ViewModel профиля пользователя»

**Короткий девиз 2026**:
> `class` — это когда тебе нужен **общий mutable state**, **наследование**, **идентичность** или **deinit**.  
> В 2026 году используй `final class`, `@MainActor`, `weak`/`unowned`, dependency injection и избегай ненужного наследования.  
> Всё остальное (модели данных, immutable state) — лучше делать `struct`.
