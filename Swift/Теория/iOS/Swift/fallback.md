**`Fallback`** в контексте [[iOS]] и [[Swift]] — это **резервный (запасной) механизм аутентификации**, который активируется, когда основной метод (обычно биометрия — [[FaceID]] / Touch ID) недоступен, не сработал или пользователь явно выбрал его.

Это **не просто кнопка**, а важная часть **LocalAuthentication** фреймворка, которая обеспечивает:

- безопасность (если биометрия сломана или отключена — есть запасной путь)
- удобство (пользователь может быстро переключиться на пароль/PIN)
- соответствие требованиям Apple Human Interface Guidelines

### 1. Когда и почему срабатывает fallback

| Ситуация                                      | Fallback активируется? | Что видит пользователь |
|-----------------------------------------------|-------------------------|-------------------------|
| Устройство **не поддерживает** биометрию     | Да                      | Только пароль/PIN       |
| Биометрия **отключена** в настройках         | Да                      | Только пароль/PIN       |
| Пользователь **5 раз ввёл неверно** биометрию| Да                      | Только пароль/PIN       |
| Пользователь **явно нажал** кнопку fallback  | Да                      | Переход к паролю/PIN    |
| Устройство в **режиме Guided Access**        | Да                      | Только пароль/PIN       |
| Биометрия **временно заблокирована** (после нескольких неудач) | Да              | Только пароль/PIN       |

### 2. Самый современный и рекомендуемый паттерн 2026 года

```swift
import LocalAuthentication
import UIKit

final class AuthManager {
    
    private let context = LAContext()
    private var error: NSError?
    
    /// Пытается аутентифицироваться через биометрию с fallback на пароль/PIN
    func authenticate(reason: String, fallbackTitle: String? = nil) async -> Result<Void, AuthError> {
        context.localizedFallbackTitle = fallbackTitle ?? "Ввести пароль"
        
        // Проверяем, доступна ли биометрия вообще
        var authError: NSError?
        let canUseBiometry = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &authError)
        
        // Если биометрия недоступна — сразу идём на fallback (пароль/PIN)
        let policy: LAPolicy = canUseBiometry ? .deviceOwnerAuthenticationWithBiometrics : .deviceOwnerAuthentication
        
        do {
            try await context.evaluatePolicy(policy, localizedReason: reason)
            return .success(())
        } catch let laError as LAError {
            switch laError.code {
            case .userCancel:
                return .failure(.userCancelled)
            case .userFallback:
                return .failure(.userChoseFallback)
            case .authenticationFailed:
                return .failure(.biometryFailed)
            case .biometryNotAvailable:
                return .failure(.biometryNotAvailable)
            case .biometryNotEnrolled:
                return .failure(.biometryNotEnrolled)
            case .biometryLockout:
                return .failure(.biometryLockedOut)
            default:
                return .failure(.unknown(laError))
            }
        } catch {
            return .failure(.system(error))
        }
    }
}

enum AuthError: Error, LocalizedError {
    case userCancelled
    case userChoseFallback
    case biometryFailed
    case biometryNotAvailable
    case biometryNotEnrolled
    case biometryLockedOut
    case system(Error)
    
    var errorDescription: String? {
        switch self {
        case .userCancelled:        return "Аутентификация отменена"
        case .userChoseFallback:    return "Выбран вход по паролю"
        case .biometryFailed:       return "Не удалось распознать лицо/отпечаток"
        case .biometryNotAvailable: return "Биометрия недоступна на этом устройстве"
        case .biometryNotEnrolled:  return "Биометрия не настроена"
        case .biometryLockedOut:    return "Биометрия временно заблокирована"
        case .system(let error):    return error.localizedDescription
        }
    }
}
```

### 3. Как правильно показать fallback в UI (рекомендации Apple 2026)

```swift
class LoginViewController: UIViewController {
    
    private let authManager = AuthManager()
    
    @IBAction func loginTapped(_ sender: UIButton) {
        Task {
            let result = await authManager.authenticate(
                reason: "Войдите, чтобы увидеть свои заметки",
                fallbackTitle: "Ввести пароль"
            )
            
            switch result {
            case .success:
                navigateToMainScreen()
            case .failure(let error):
                switch error {
                case .userChoseFallback:
                    showPasswordLogin()
                case .biometryLockedOut:
                    showBiometryLockedAlert()
                default:
                    showErrorAlert(error.localizedDescription ?? "Неизвестная ошибка")
                }
            }
        }
    }
    
    private func showPasswordLogin() {
        // Показываем экран ввода пароля / PIN
        let alert = UIAlertController(title: "Вход по паролю", message: nil, preferredStyle: .alert)
        // ...
        present(alert, animated: true)
    }
}
```

### 4. Лучшие практики fallback в 2026

- **Всегда** задавай `localizedFallbackTitle` — это текст кнопки fallback
- **Не скрывай** кнопку fallback — это нарушает HIG (Human Interface Guidelines)
- **Обрабатывай** `.userFallback` отдельно — пользователь **явно выбрал** пароль
- **Используй** `.deviceOwnerAuthentication` вместо `.deviceOwnerAuthenticationWithBiometrics`, если хочешь **всегда** давать fallback
- **В [[async]]** — оборачивай в `Task` и обновляй UI через `await MainActor.run`
- **[[Swift]] 6 strict concurrency** — `LAContext` безопасен, но используй `@MainActor` для UI-обновлений
- **Документируйте** — пиши комментарий «fallback на пароль при неудачной биометрии»

**Короткий девиз 2026**:
> Fallback — это **не ошибка**, а **нормальный путь**.  
> В 2026 году:  
> - всегда показывай кнопку fallback  
> - явно обрабатывай `.userFallback`  
> - используй `.deviceOwnerAuthentication` для максимальной надёжности  
> - обновляй UI только на главном акторе
