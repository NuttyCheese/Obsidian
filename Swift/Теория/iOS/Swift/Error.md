**`Error`** — это **протокол** в Swift, который маркирует тип как **возможную ошибку**, которую можно **выбрасывать** (`throw`) из функций с модификатором `throws` и **ловить** в блоках `do-catch` или через `Result`.

> Проще говоря: если тип соответствует `Error` → его можно `throw`, а потом поймать и обработать.

### 1. Почему именно протокол, а не класс или struct?

`Error` — это **маркерный протокол** (marker protocol) без требований (пустой).  
Это сделано намеренно:

- Любой тип (struct, enum, class) может быть ошибкой  
- Не нужно наследоваться от конкретного класса (гибкость)  
- Минимальные накладные расходы  
- Легко комбинировать с `Result`, `throws`, `async throws`

```swift
public protocol Error : Sendable { }
```

(в Swift 6+ `Error` наследует `Sendable` → все ошибки потокобезопасны)

### 2. Самые популярные способы создания Error (2026 стандарт)

#### Вариант 1 — enum с cases (99% всех ошибок в реальных проектах)

```swift
enum NetworkError: Error {
    case offline
    case timeout
    case server(statusCode: Int, message: String?)
    case decodingFailed(underlying: DecodingError)
}
```

#### Вариант 2 — enum с associated values (самый мощный)

```swift
enum AuthError: Error {
    case wrongCredentials(username: String)
    case accountLocked(until: Date)
    case biometryFailed(reason: BiometryFailureReason)
}
```

#### Вариант 3 — struct с дополнительными полями (когда нужна богатая информация)

```swift
struct ValidationError: Error, CustomStringConvertible {
    let field: String
    let rule: String
    let value: String?
    
    var description: String {
        "Неверное значение в поле '\(field)': \(rule)"
    }
}
```

#### Вариант 4 — использование Foundation-ошибок (NSError)

```swift
let error = NSError(domain: "com.myapp.auth", code: -1001, userInfo: [
    NSLocalizedDescriptionKey: "Нет соединения с сервером"
])
throw error
```

### 3. Полный пример реального использования (2026 стиль — async + do-catch + Result)

```swift
@MainActor
class AuthViewModel: ObservableObject {
    
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    enum AuthError: Error, LocalizedError {
        case wrongCredentials
        case networkUnavailable
        case server(statusCode: Int)
        
        var errorDescription: String? {
            switch self {
            case .wrongCredentials:    return "Неверный логин или пароль"
            case .networkUnavailable:  return "Нет соединения с интернетом"
            case .server(let code):    return "Ошибка сервера (\(code))"
            }
        }
    }
    
    func login(email: String, password: String) async {
        isLoading = true
        errorMessage = nil
        
        do {
            let user = try await authService.login(email: email, password: password)
            await handleSuccessfulLogin(user)
        } catch AuthError.wrongCredentials {
            errorMessage = "Неверный логин или пароль"
        } catch AuthError.networkUnavailable {
            errorMessage = "Нет интернета. Проверьте соединение"
        } catch let error as NSError where error.code == NSURLErrorNotConnectedToInternet {
            errorMessage = "Нет соединения с интернетом"
        } catch {
            errorMessage = "Произошла неизвестная ошибка: \(error.localizedDescription)"
            // Логируем в Crashlytics / Sentry
            logger.error("Login failed: \(error)")
        }
        
        isLoading = false
    }
}
```

### 4. Лучшие практики Error в Swift 2026

- **Предпочитайте enum** — он самый читаемый и поддерживает exhaustive switch  
- **Добавляйте LocalizedError** — чтобы `error.localizedDescription` был понятен пользователю  
- **Используйте associated values** для передачи контекста (код ошибки, поле, сообщение)  
- **Ловите конкретные ошибки первыми** — порядок catch имеет значение  
- **Оставляйте универсальный catch последним** — для неизвестных ошибок  
- **В async** — всегда `try await` внутри `do-catch`  
- **Для UI** — показывайте `error.localizedDescription` или кастомное сообщение  
- **Логируйте** неизвестные ошибки (os_log, Crashlytics, Sentry)  
- **Swift 6 strict concurrency** — все типы `Error` должны быть `Sendable` (enum и struct по умолчанию)  
- **Документируйте** — пиши комментарий «AuthError.wrongCredentials — неверные логин/пароль»

**Короткий девиз 2026**:
> `Error` — это маркер: «это можно throw и поймать в catch».  
> В 2026 году:  
> - enum + associated values — основной способ  
> - LocalizedError — для понятных сообщений пользователю  
> - конкретные catch → сверху, универсальный catch → снизу  
> - логируй неизвестные ошибки  
> Это **единственный безопасный** способ обрабатывать `throws` и `async throws`.

Удачи с надёжной и понятной обработкой ошибок в твоём коде! 🛡️