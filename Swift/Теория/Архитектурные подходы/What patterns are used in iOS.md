Вот **полный, подробный и максимально актуальный** (на 2026 год) обзор самых распространённых **паттернов проектирования** в **iOS-разработке на Swift** — с реальными примерами, статусом использования, рекомендациями и сравнением с современными альтернативами.

### Топ-9 самых живых паттернов в iOS 2026 году

| №   | Паттерн                                                | Категория [[GoF]] | Актуальность 2026 | Основные места использования в [[iOS]]                  | Современная идиоматичная реализация      | Рекомендация          |
| --- | ------------------------------------------------------ | ----------------- | ----------------- | ------------------------------------------------------- | ---------------------------------------- | --------------------- |
| 1   | **[[MVC (Model-View-Controller) Architecture\|MVC]]**  | Архитектура       | ★★★★★             | [[UIKit]] ([[UIViewController]])                        | UIView + ViewModel + Model               | Базовый, везде        |
| 2   | **[[MVVM (Model-View-ViewModel) Architecture\|MVVM]]** | Архитектура       | ★★★★★             | [[SwiftUI]] + UIKit                                     | @Observable / @Published + ViewModel     | Основной в новом коде |
| 3   | **[[Delegate]]**                                       | Поведенческий     | ★★★★★             | [[UITableView]], [[UITextField]], [[URLSession]] и т.д. | [[Protocol]] + [[weak]] ссылка           | Классика iOS          |
| 4   | **[[Strategy]]**                                       | Поведенческий     | ★★★★★             | Сортировка, валидация, оплата, анимации                 | enum + switch / протокол + реализации    | Очень часто           |
| 5   | **[[Decorator]]**                                      | Структурный       | ★★★★★             | Сетевые запросы (лог, retry, cache)                     | Функциональные обёртки / extension       | Очень часто           |
| 6   | **[[Adapter]]**                                        | Структурный       | ★★★★★             | Интеграция старых SDK, разные [[API]]                   | extension + протокол                     | Очень часто           |
| 7   | **[[Builder]]**                                        | Порождающий       | ★★★★★             | Сложные модели, запросы, конфиги                        | Fluent chain / Result Builder            | Очень часто           |
| 8   | **[[Observer]]**                                       | Поведенческий     | ★★★★★             | SwiftUI, [[Combine]], [[NotificationCenter]]            | @Observable / @Published / AsyncSequence | Встроено в язык       |
| 9   | **[[Facade]]**                                         | Структурный       | ★★★★☆             | Обёртка над сложными SDK ([[Firebase]], Amplitude)      | Отдельный сервис-класс                   | Часто                 |

### Подробно по каждому паттерну (с примерами 2026 года)

#### 1. MVC (Model-View-Controller) — всё ещё живой, но сильно эволюционировал

```swift
// Model
struct User {
    let id: UUID
    let name: String
}

// View
class UserCell: UITableViewCell {
    func configure(with user: User) {
        textLabel?.text = user.name
    }
}

// Controller
class UsersViewController: UIViewController, UITableViewDataSource {
    private var users: [User] = []
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        users.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "UserCell", for: indexPath) as! UserCell
        cell.configure(with: users[indexPath.row])
        return cell
    }
}
```

**2026 статус**:  
- UIKit → MVC всё ещё основной  
- SwiftUI → MVVM / [[MVI (Model-View-Intent) Architecture|MVI]] / [[TCA]] полностью вытеснили классический MVC  
- **Массивный ViewController** → антипаттерн (Massive View Controller)

#### 2. MVVM (Model-View-ViewModel) — основной паттерн 2026 года

```swift
@Observable
class UsersViewModel {
    var users: [User] = []
    var isLoading = false
    var error: Error?
    
    private let repository: any UserRepository
    
    init(repository: any UserRepository) {
        self.repository = repository
    }
    
    func load() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            users = try await repository.fetchUsers()
        } catch {
            self.error = error
        }
    }
}

// SwiftUI View
struct UsersView: View {
    @State private var vm = UsersViewModel(repository: APIRepository())
    
    var body: some View {
        List(vm.users) { user in
            Text(user.name)
        }
        .overlay {
            if vm.isLoading { ProgressView() }
        }
        .task { await vm.load() }
    }
}
```

**2026 тренд**:  
`@Observable` (Swift 5.9+) почти полностью вытеснил `@ObservableObject` + `objectWillChange`

#### 3. [[Delegate]] — классика iOS, никуда не делась

```swift
protocol UserSelectionDelegate: AnyObject {
    func didSelectUser(_ user: User)
}

class UserListViewController: UIViewController {
    weak var delegate: UserSelectionDelegate?
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let user = users[indexPath.row]
        delegate?.didSelectUser(user)
    }
}
```

**2026 тренд**:  
- **weak** ссылка обязательна  
- часто заменяется на **closure** или **@Observable** / **AsyncSequence**

#### 4. Strategy — один из самых живых паттернов

(см. предыдущий подробный ответ по Strategy)

#### 5. Decorator — очень часто в сетевом слое

(см. предыдущий подробный ответ по Decorator)

#### 6. Adapter — почти каждый день

```swift
// Старый API возвращает NSDictionary
protocol ModernUserService {
    func fetchUsers() async throws -> [User]
}

struct LegacyAdapter: ModernUserService {
    let legacyService: LegacyUserService
    
    func fetchUsers() async throws -> [User] {
        let dicts = try await legacyService.fetchUsersLegacy()
        return dicts.compactMap { User(from: $0) }
    }
}
```

#### 7. Builder — must-have для сложных моделей

(см. предыдущий подробный ответ по Builder)

#### 8. Observer — уже встроен в язык

(см. предыдущий подробный ответ по Observer)

#### 9. Facade — обёртка над сложными SDK

```swift
// Facade над Firebase + Analytics + Crashlytics
actor AnalyticsFacade {
    func track(_ event: String, parameters: [String: Any]? = nil) {
        // Firebase
        // Amplitude
        // Sentry
    }
    
    func logError(_ error: Error) {
        // Crashlytics
        // Sentry
    }
}
```

### 6. Топ-5 самых используемых паттернов в iOS 2026

1. **MVVM + @Observable** — основной для [[SwiftUI]]/[[UIKit]]  
2. **[[Delegate]] + [[closure]]** — классика [[UIKit]]  
3. **[[Strategy]]** — валидация, сортировка, оплата, обработка ошибок  
4. **[[Decorator]]** — сетевые middleware (лог, retry, cache)  
5. **[[Builder]]** — сложные модели, конфиги, запросы

### 7. Лучшие практики паттернов в Swift 2026

- **Протоколы** — маленькие, узкие, конкретные  
- **actor** — для всего изменяемого состояния  
- **@Observable** / `@Published` — для UI-реактивности  
- **Dependency Injection** — через init или @Environment  
- **Async/await** — основной способ асинхронности  
- **Swift 6 strict concurrency** — все паттерны должны быть Sendable-safe  
- **Тестирование** — маленькие протоколы + моки = идеальные unit-тесты  
- **Документируйте** — пишите в документации протокола/класса «реализует паттерн X»

**Короткий девиз 2026**:
> «В 2026 году лучшие iOS-приложения строятся на **MVVM + @Observable + Strategy + Decorator + Builder + DI**.  
> Классические GoF-паттерны живы, но выглядят по-новому — через протоколы, actor, extension и async/await.»
