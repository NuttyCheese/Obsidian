**LAContext** — это основной класс фреймворка **[[LocalAuthentication]]** (доступен с [[iOS]] 8, 2014 год), который отвечает за **всю работу с биометрической и парольной аутентификацией** на устройстве Apple.

С помощью одного объекта `LAContext` вы можете:

- проверить, поддерживает ли устройство биометрию вообще
- определить, какая именно биометрия доступна ([[FaceID]] / [[TouchID]])
- выполнить аутентификацию (с биометрией или с fallback на пароль)
- узнать состояние биометрии (заблокирована ли, настроена ли)
- управлять контекстом (invalidate, fallback title и т.д.)

### Ключевые свойства и методы LAContext (актуально на 2026 год)

| Свойство / Метод                             | Тип / Возвращает                              | Что делает / зачем нужен                                | Самый частый сценарий                        |
| -------------------------------------------- | --------------------------------------------- | ------------------------------------------------------- | -------------------------------------------- |
| `biometryType`                               | [[LABiometryType]] (.none, .touchID, .faceID) | Какой тип биометрии доступен на устройстве              | Определить Face ID или Touch ID              |
| `canEvaluatePolicy(_:error:)`                | `Bool`                                        | Можно ли выполнить аутентификацию с данной политикой    | Обязательная проверка перед вызовом evaluate |
| `evaluatePolicy(_:localizedReason:reply:)`   | асинхронный callback                          | Запуск аутентификации (биометрия +/– пароль)            | Основной метод входа                         |
| `evaluatePolicy(_:localizedReason:)` (async) | `async throws -> Bool` (iOS 15+)              | Современная async-версия                                | [[Swift Concurrency]] / [[SwiftUI]]          |
| `isBiometryAvailable`                        | `Bool` (вычисляемое)                          | Быстрая проверка наличия биометрии (Touch ID / Face ID) | Упрощённая проверка                          |
| `biometryLockout`                            | [[Bool]] (iOS 11+)                            | Биометрия временно заблокирована (5+ неудач)            | Показать "Введите пароль"                    |
| `localizedFallbackTitle`                     | [[String]]`?`                                 | Текст кнопки fallback (вместо "Ввести пароль")          | Кастомизация UX                              |
| `localizedCancelTitle`                       | `String?` (iOS 10+)                           | Текст кнопки "Отмена"                                   | Локализация                                  |
| `interactionNotAllowed`                      | `Bool`                                        | Запрещено ли взаимодействие (редко)                     | Enterprise-ограничения                       |
| `invalidate()`                               | `Void`                                        | Сбрасывает состояние контекста (после неудачи)          | После ошибки                                 |
| `setCredential(_:type:)`                     | `Bool`                                        | Установка кастомного credential (редко)                 | Enterprise / MDM                             |

### Полный современный пример (2026 — [[async]]/[[await]] + обработка всех ошибок)

```swift
import LocalAuthentication

/// Проверка доступности биометрии
func checkBiometry() async -> BiometryStatus {
    let context = LAContext()
    var error: NSError?
    
    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
        switch context.biometryType {
        case .faceID:    return .faceID
        case .touchID:   return .touchID
        case .none:      return .none
        @unknown default: return .unknown
        }
    }
    
    return .unavailable(error?.localizedDescription ?? "Неизвестно")
}

/// Самый рекомендуемый способ аутентификации (с fallback на пароль)
func authenticate(
    reason: String = "Вход в защищённый раздел",
    fallbackTitle: String? = "Ввести пароль"
) async -> Result<Bool, LAError> {
    
    let context = LAContext()
    context.localizedFallbackTitle = fallbackTitle
    
    do {
        let success = try await context.evaluatePolicy(
            .deviceOwnerAuthentication,
            localizedReason: reason
        )
        return .success(success)
    } catch let error as LAError {
        return .failure(error)
    } catch {
        return .failure(LAError(.appCancel))
    }
}

// Использование в SwiftUI / async контексте
Task {
    let result = await authenticate(reason: "Подтвердите доступ к кошельку")
    
    switch result {
    case .success(true):
        print("Успешно вошли")
        // переход на защищённый экран
        
    case .success(false):
        print("Аутентификация отклонена")
        
    case .failure(let laError):
        switch laError.code {
        case .authenticationFailed:
            await showAlert("Неверный отпечаток / лицо")
        case .userCancel:
            // ничего не делаем
            break
        case .userFallback:
            // пользователь выбрал пароль → можно показать поле ввода пароля
            await showPasscodeScreen()
        case .biometryNotAvailable:
            await showAlert("Биометрия недоступна на этом устройстве")
        case .biometryLockout:
            await showAlert("Биометрия заблокирована. Введите пароль.")
        case .appCancel:
            // приложение отменило запрос
            break
        default:
            await showAlert("Ошибка: \(laError.localizedDescription)")
        }
    }
}
```

### Обработка всех основных ошибок [[LAError]] (полный список 2026)

```swift
switch error.code {
case .authenticationFailed:     // -1   — неверный отпечаток / лицо
case .userCancel:               // -2   — нажата "Отмена"
case .userFallback:             // -3   — выбрана кнопка "Ввести пароль"
case .systemCancel:             // -4   — система прервала (звонок, Siri и т.д.)
case .biometryNotAvailable:     // -6   — устройство без биометрии
case .biometryNotEnrolled:      // -7   — биометрия не настроена
case .biometryLockout:          // -8   — 5+ неудач → временная блокировка
case .appCancel:                // -9   — приложение само отменило запрос
case .invalidContext:           // -10  — контекст недействителен
case .notInteractive:           // -1004 — UI не может быть показан
@unknown default:
    print("Неизвестный код ошибки")
}
```

### Лучшие практики [[LocalAuthentication]] / LAContext в 2026

- **Всегда** используйте **`.deviceOwnerAuthentication`** — это стандартный и удобный вариант (биометрия + пароль)  
- **Предпочитайте** **async/await** версию `evaluatePolicy` — код чище и современнее  
- **Обязательно** обрабатывайте **все** случаи `LAError` — особенно `.biometryLockout` и `.userFallback`  
- **Перед вызовом** всегда проверяйте `canEvaluatePolicy` — это предотвращает лишние алерты  
- **Для максимальной безопасности** (банки, крипто) — используйте `.deviceOwnerAuthenticationWithBiometrics`  
- **Для iPad** — Touch ID есть только на старых моделях → всегда проверяйте `biometryType`  
- **Privacy Manifest** — обязателен с iOS 17+ при использовании LocalAuthentication  
- **Документируйте** — пишите комментарий «LAContext — аутентификация через биометрию (Face ID / Touch ID) с fallback на пароль»

**Короткий итог 2026**:
> LAContext — это **центральный класс** для работы с биометрией и паролем в iOS.  
> В 2026 году:  
> - основной метод — `evaluatePolicy(.deviceOwnerAuthentication, ...)`  
> - проверка — `canEvaluatePolicy` + `biometryType` (.faceID / .touchID)  
> - обработка ошибок — **обязательна** (особенно `.biometryLockout`, `.userFallback`)  
> - async/await версия — стандарт для нового кода  
> Это **единственный** и **самый безопасный** способ реализовать Face ID / Touch ID / пароль в приложении.
