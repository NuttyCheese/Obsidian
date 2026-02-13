### 1. Что такое DRY и почему он всё ещё важен в 2026 году

**DRY** — один из самых старых и фундаментальных принципов разработки, сформулированный в книге «The Pragmatic Programmer» (1999).

Классическая формулировка:

> «Каждая часть знания должна иметь единственное, однозначное и авторитетное представление в системе.»

Перевод на язык [[Swift]] 2026:

- **Не копируй и не вставляй** один и тот же код в разные места  
- Если логика повторяется ≥ 2 раз — её **надо** вынести в отдельную сущность  
- Изменение одной вещи должно происходить **в одном месте**

**Почему DRY в 2026 году стал ещё критичнее**:

- **Swift 6 strict concurrency** → дублированный код = дублированные data race и ошибки изоляции  
- **[[TCA]] / Composable Architecture** → DRY лежит в основе reducer’ов и эффектов  
- **[[Clean Swift (VIP) Architecture|Clean Swift]] / [[VIPER Architecture|VIPER]] / [[MVVM (Model-View-ViewModel) Architecture|MVVM]]-[[Coordinator]]** → DRY обязателен для UseCase / Interactor / Repository  
- **AI-генерация кода** (Copilot, Cursor, [[Xcode]] 18+ AI) → генерирует много похожего кода → без DRY проект превращается в кашу за 2–3 месяца  
- **Команды 5–20+ человек** → дублированный код = бесконечные merge-конфликты и регрессии

**Самый короткий и честный девиз 2026**:
> «Если ты видишь один и тот же код в двух местах — один из них уже устарел.  
> DRY — это не про лень, а про защиту от человеческого фактора и будущих изменений.»

### 2. Самые частые нарушения DRY в iOS-приложениях 2026 года

| Нарушение DRY                                      | Почему это плохо в 2026 году                          | Как исправить (DRY-подход) |
|-----------------------------------------------------|--------------------------------------------------------|-----------------------------|
| Копирование валидации форм в 5 ViewController’ах    | Изменение правила → править в 5 местах → регрессия     | Одна `FormValidator` структура |
| Дублирование обработки ошибок в 10 сетевых вызовах  | Новый тип ошибки → 10 мест правки                      | Один `ErrorHandler` / `NetworkErrorMapper` |
| Повторяющийся код загрузки изображения в 8 местах   | Переход на Kingfisher / AsyncImage → 8 правок          | Одна `ImageLoadingService` / extension |
| Копирование форматирования даты в 15 ViewModel’ах   | Изменение формата → 15 мест правки                     | Один `DateFormatterService` / extension Date |
| Дублирование логики «показать алерт ошибки»        | Новый дизайн алерта → править везде                    | Один `AlertPresenter` / `ErrorAlertBuilder` |
| Повторяющийся код авторизации в 5 местах           | Новый токен-механизм → править в 5 местах              | Один `AuthService` / `AuthManager` (actor) |

### 3. Самые популярные и рекомендуемые реализации DRY в Swift 2026

#### Паттерн 1 — Вынос повторяющегося кода в extension (самый простой и частый)

```swift
// До (дублирование в 10 местах)
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
dateFormatter.timeStyle = .short
dateFormatter.locale = Locale(identifier: "ru_RU")
let formatted = dateFormatter.string(from: Date())

// После (одно место)
extension Date {
    static let ruMediumShort: DateFormatter = {
        let df = DateFormatter()
        df.dateStyle = .medium
        df.timeStyle = .short
        df.locale = Locale(identifier: "ru_RU")
        return df
    }()
    
    var ruMediumShort: String {
        Date.ruMediumShort.string(from: self)
    }
}

// Использование в любом месте
let text = Date().ruMediumShort
```

#### Паттерн 2 — Вынос повторяющейся логики в протокол + default-реализация

```swift
protocol ErrorPresentable {
    func showError(_ error: Error, retryAction: (() -> Void)?)
}

extension ErrorPresentable where Self: UIViewController {
    func showError(_ error: Error, retryAction: (() -> Void)? = nil) {
        let alert = UIAlertController(
            title: "Ошибка",
            message: error.localizedDescription,
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        
        if let retry = retryAction {
            alert.addAction(UIAlertAction(title: "Повторить", style: .default) { _ in retry() })
        }
        
        present(alert, animated: true)
    }
}

// Теперь любой UIViewController может использовать метод без дублирования
class ProfileViewController: UIViewController, ErrorPresentable {
    func load() async {
        do {
            // ...
        } catch {
            showError(error) { [weak self] in
                Task { await self?.load() }
            }
        }
    }
}
```

#### Паттерн 3 — DRY через actor + TaskLocal (очень мощно в 2026)

```swift
@TaskLocal
static var currentRequestID: String?

actor APIService {
    func fetchUsers() async throws -> [User] {
        let requestID = TaskLocal.currentRequestID.currentValue ?? UUID().uuidString
        
        var request = URLRequest(url: usersURL)
        request.setValue(requestID, forHTTPHeaderField: "X-Request-ID")
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode([User].self, from: data)
    }
}

// В любой точке приложения
await TaskLocal.currentRequestID.withValue("req-abc123") {
    let users = try await api.fetchUsers()
    // requestID автоматически попал в заголовок
}
```

### 4. Визуальная схема DRY (2026 стиль)

```mermaid
flowchart TD
    DuplicatedCode["Дублированный код в 7 местах"] --> Bad["Нарушение DRY<br>→ 7 мест правки при изменении"]
    
    Refactored["Выносим в одно место"] --> Good["Одна причина изменения<br>→ меняем только здесь"]
    
    Refactored --> Extension["extension Date"]
    Refactored --> ProtocolDefault["extension ErrorPresentable"]
    Refactored --> Actor["actor AuthService"]
    Refactored --> TaskLocal["@TaskLocal currentRequestID"]
    
    Good --> "Легче тестировать"
    Good --> "Меньше багов"
    Good --> "Быстрее рефакторинг"
```

### 5. Лучшие практики DRY в Swift 2026

- **Правило трёх** — увидел код 3 раза → выноси в отдельную функцию/класс/extension  
- **extension** — самый мощный инструмент DRY в Swift (на протоколы, на типы, на Any)  
- **actor** — идеально для DRY в многопоточном коде (один источник истины)  
- **TaskLocal** — DRY для передачи контекста (request ID, user ID, trace ID) без параметров  
- **Result Builder** — DRY для DSL-подобных конструкций (SwiftUI, TCA, Kingfisher, Alamofire)  
- **Не бойтесь** создавать маленькие вспомогательные типы (Formatter, Validator, Mapper, Builder)  
- **Swift 6 strict concurrency** — DRY + actor + TaskLocal = почти 100% отсутствие data race  
- **Тестирование** — один мок / stub на всю команду вместо дублирования в каждом тесте  
- **Документируйте** — пишите в документации «используется в 12 местах — см. extension»

**Короткий девиз 2026**:
> «DRY в 2026 году — это когда ты пишешь код так, чтобы изменение одной вещи происходило в одном месте.  
> Без DRY в Swift 6+ проект через 1–2 года превращается в спагетти, которое невозможно поддерживать.»

Удачи с чистым, DRY и современным кодом в Swift! 🧹