**Touch ID** — это биометрическая технология Apple для аутентификации по отпечатку пальца.  
Она появилась в iPhone 5s (2013) и до сих пор остаётся актуальной в 2026 году на множестве устройств (особенно на iPhone SE, iPad Air/Mini, старых iPad Pro и MacBook с Touch ID).

Touch ID используется для:
- разблокировки устройства
- авторизации в приложениях
- подтверждения платежей (Apple Pay)
- входа в защищённые разделы (пароли, банковские приложения, 1Password и т.д.)

В [[iOS]]/macOS доступна исключительно через фреймворк **[[LocalAuthentication]]** ([[LAContext]]).

### Ключевые факты о Touch ID в 2026 году

| Характеристика                     | Значение / Особенность                                      | Важные замечания 2026 |
|------------------------------------|-------------------------------------------------------------|-----------------------|
| Точность                           | 1 к 50 000 (против 1 к 1 000 000 у Face ID)                 | Всё ещё очень безопасно |
| Скорость                           | ~0.2–0.4 секунды                                            | Быстрее Face ID в некоторых сценариях (особенно в темноте) |
| Условия работы                     | Требуется чистый палец, может не работать с мокрыми/грязными руками | Не работает в перчатках |
| Количество отпечатков              | До 5 отпечатков на устройство                               | Семейный доступ — нет |
| Резервный способ                   | Пароль / код доступа (обязателен)                           | После 5 неудач → требует пароль |
| Поддержка устройств (2026)         | iPhone 5s – 8, iPhone SE (1–3), многие iPad, MacBook Pro/Air с Touch ID | Нет на iPhone X и новее (только Face ID) |
| Face ID vs Touch ID                | Face ID быстрее и точнее в большинстве случаев              | Touch ID выигрывает в темноте и при мокрых руках |

### Основные политики аутентификации в LocalAuthentication

| Политика                                      | Описание                                                                 | Когда использовать в 2026 |
|-----------------------------------------------|--------------------------------------------------------------------------|----------------------------|
| `.deviceOwnerAuthenticationWithBiometrics`    | Только Touch ID / Face ID (без пароля как fallback)                      | Максимальная безопасность (банки, крипто-кошельки) |
| `.deviceOwnerAuthentication`                  | Touch ID / Face ID + пароль как fallback                                 | Самый популярный и удобный выбор (99% приложений) |
| `.applicationPassword`                        | Только пароль приложения (без биометрии)                                 | Редко, для дополнительной защиты |

### Полный современный пример (2026 стиль — [[async]]/[[await]] + обработка всех ошибок)

```swift
import LocalAuthentication

/// Проверяет наличие и тип биометрии
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
    } else {
        return .unavailable(error?.localizedDescription ?? "Неизвестная причина")
    }
}

/// Аутентификация с fallback на пароль (самый рекомендуемый вариант)
func authenticate(
    reason: String = "Вход в приложение",
    fallbackTitle: String? = "Использовать пароль"
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
        // Редкие не-LA ошибки
        return .failure(LAError(.appCancel))
    }
}

// Пример использования в SwiftUI / async контексте
Task {
    switch await authenticate(reason: "Подтвердите вход") {
    case .success(true):
        print("Успешная аутентификация")
        // Переход на главный экран
        
    case .success(false):
        print("Аутентификация отклонена (но без ошибки)")
        
    case .failure(let laError):
        switch laError.code {
        case .authenticationFailed:
            print("Неверный отпечаток / лицо")
        case .userCancel:
            print("Пользователь отменил")
        case .userFallback:
            print("Перешёл на пароль")
        case .biometryNotAvailable:
            print("Биометрия недоступна")
        case .biometryLockout:
            print("Биометрия заблокирована — требуется пароль")
        case .appCancel:
            print("Приложение отменило запрос")
        default:
            print("Другая ошибка:", laError.localizedDescription)
        }
    }
}
```

### Обработка всех основных ошибок LAError (полный справочник 2026)

```swift
switch error.code {
case .authenticationFailed:     // 3   — не распознан отпечаток/лицо
case .userCancel:               // -2  — отмена (кнопка "Отмена")
case .userFallback:             // -3  — выбрал "Ввести пароль"
case .systemCancel:             // -4  — система отменила (например, входящий звонок)
case .biometryNotAvailable:     // -6  — Touch ID / Face ID не поддерживается устройством
case .biometryNotEnrolled:      // -7  — биометрия не настроена
case .biometryLockout:          // -8  — слишком много неудач → временная блокировка
case .appCancel:                // -9  — приложение отменило запрос
case .invalidContext:           // -10 — контекст недействителен (редко)
case .notInteractive:           // -1004 — UI не может быть показан (редко)
@unknown default:
    print("Неизвестный код ошибки")
}
```

### Лучшие практики LocalAuthentication / Touch ID в 2026 году

- **Предпочитайте** `.deviceOwnerAuthentication` — биометрия + пароль как запасной вариант  
- **Используйте** `async/await` версию `evaluatePolicy` — современный и чистый код  
- **Всегда** обрабатывайте **все** случаи `LAError` — особенно `.biometryLockout` и `.userFallback`  
- **Перед вызовом** всегда проверяйте `canEvaluatePolicy` — иначе получите краш или неожиданное поведение  
- **Для банков/крипто** — можно использовать `.deviceOwnerAuthenticationWithBiometrics` (без пароля)  
- **Для iPad** — Touch ID есть только на старых моделях (iPad Air 2–4, iPad mini 4–5) — проверяйте `biometryType`  
- **Privacy Manifest** — обязателен с iOS 17+ при использовании LocalAuthentication  
- **Документируйте** — пишите комментарий «Аутентификация через Touch ID / Face ID с fallback на пароль (deviceOwnerAuthentication)»

**Короткий итог 2026**:
> Touch ID — это **надёжная** и **быстрая** биометрия по отпечатку пальца (1 к 50 000).  
> В 2026 году:  
> - основной способ — `LAContext.evaluatePolicy(.deviceOwnerAuthentication, ...)`  
> - проверка доступности — `canEvaluatePolicy` + `biometryType == .touchID`  
> - обработка ошибок — **обязательна** (особенно `.biometryLockout`, `.userFallback`)  
> Это **классический** и **до сих пор широко используемый** способ биометрической аутентификации на множестве устройств Apple.
