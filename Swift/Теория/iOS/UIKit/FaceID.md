**Face ID** — это технология биометрической аутентификации по лицу, разработанная Apple.  
Она появилась в iPhone X (2017) и с тех пор стала основным способом разблокировки устройств, подтверждения платежей и входа в приложения.

В 2025–2026 годах Face ID остаётся **самым безопасным** и **самым быстрым** способом биометрической аутентификации на устройствах Apple (лучше Touch ID по точности и скорости в большинстве сценариев).

### Ключевые характеристики Face ID (актуально на 2026 год)

| Характеристика                     | Значение / Особенность                                      | Важные замечания 2026 |
|------------------------------------|-------------------------------------------------------------|-----------------------|
| Технология                         | TrueDepth-камера (инфракрасная точечная проекция + ИИ)      | 30 000+ невидимых точек |
| Точность                           | 1 к 1 000 000 (против 1 к 50 000 у Touch ID)                | Практически невозможно обмануть фото/маской |
| Скорость                           | ~0.3–0.5 секунды                                            | Быстрее Touch ID в реальных условиях |
| Условия работы                     | Работает в темноте, с очками, шляпой, маской (с iOS 15.4+)  | Маска + очки — с 2022 года |
| Количество лиц                     | До 2 лиц (Face ID с маской — отдельная настройка)           | Семейный доступ — нет |
| Резервный способ                   | Пароль / код доступа (обязателен)                           | После 5 неудач → требует пароль |
| Поддержка устройств (2026)         | iPhone X и новее + все iPad Pro (с Face ID)                 | Нет на iPhone SE, iPad Air/Mini (2026) |

### Основные политики аутентификации в LocalAuthentication

| Политика                                      | Описание                                                                 | Когда использовать в 2026 |
|-----------------------------------------------|--------------------------------------------------------------------------|----------------------------|
| `.deviceOwnerAuthenticationWithBiometrics`    | Только Face ID / Touch ID (без пароля как fallback)                      | Максимальная безопасность (банки, крипто-кошельки) |
| `.deviceOwnerAuthentication`                  | Face ID / Touch ID + пароль как fallback                                 | Самый популярный и удобный выбор (большинство приложений) |
| `.applicationPassword`                        | Только пароль приложения (без биометрии)                                 | Редко, для дополнительной защиты |

### Самые важные и часто используемые паттерны в 2026 году

#### 1. Проверка поддержки и типа биометрии (самый частый)

```swift
import LocalAuthentication

func checkBiometryAvailability() {
    let context = LAContext()
    var authError: NSError?
    
    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &authError) {
        switch context.biometryType {
        case .faceID:
            print("Face ID доступен и поддерживается")
        case .touchID:
            print("Touch ID доступен")
        case .none:
            print("Биометрия не поддерживается")
        @unknown default:
            print("Неизвестный тип биометрии")
        }
    } else {
        print("Биометрия недоступна:", authError?.localizedDescription ?? "Неизвестная ошибка")
    }
}
```

#### 2. Современная аутентификация с fallback на пароль (золотой стандарт 2026)

```swift
func authenticateWithBiometryOrPasscode(reason: String = "Вход в приложение") async -> Bool {
    let context = LAContext()
    
    do {
        let success = try await context.evaluatePolicy(
            .deviceOwnerAuthentication,
            localizedReason: reason
        )
        return success
    } catch let error as LAError {
        switch error.code {
        case .authenticationFailed:
            print("Аутентификация не удалась")
        case .userCancel:
            print("Пользователь отменил")
        case .userFallback:
            print("Пользователь выбрал пароль")
        case .biometryNotAvailable:
            print("Биометрия недоступна")
        case .biometryLockout:
            print("Face ID временно заблокирован (слишком много неудач)")
        default:
            print("Другая ошибка:", error.localizedDescription)
        }
        return false
    } catch {
        print("Неизвестная ошибка:", error.localizedDescription)
        return false
    }
}

// Использование в SwiftUI / async контексте
Task {
    let authenticated = await authenticateWithBiometryOrPasscode()
    if authenticated {
        // вход успешен
    }
}
```

#### 3. Обработка Face ID с маской (с iOS 15.4+)

```swift
func checkFaceIDWithMaskSupport() {
    let context = LAContext()
    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: nil) &&
       context.biometryType == .faceID {
        
        // Проверка поддержки Face ID с маской
        if context.isCapabilityEnabled(.biometryCurrentSetRequiresMask) {
            print("Face ID с маской включён")
        } else {
            print("Обычный Face ID")
        }
    }
}
```

### Лучшие практики Face ID / LocalAuthentication в 2026 году

- **Используйте** `.deviceOwnerAuthentication` — это стандартный и удобный выбор (биометрия + fallback на пароль)  
- **Всегда** обрабатывайте **все** ошибки `LAError` — особенно `.biometryLockout`, `.userFallback`, `.userCancel`  
- **Для SwiftUI** — используйте `async/await` версию `evaluatePolicy` — она чище и современнее  
- **Для банков/крипто** — можно использовать `.deviceOwnerAuthenticationWithBiometrics` (без пароля как fallback)  
- **Не забывайте** проверять `canEvaluatePolicy` **перед** вызовом `evaluatePolicy`  
- **Для iPad** — Face ID есть только на iPad Pro (с 2018 года) — проверяйте `biometryType`  
- **Privacy Manifest** — обязателен с iOS 17+ при использовании LocalAuthentication  
- **Документируйте** — пишите комментарий «Аутентификация через Face ID / Touch ID с fallback на пароль (deviceOwnerAuthentication)»

**Короткий итог 2026**:
> Face ID — это **самая безопасная** биометрическая аутентификация Apple (1 к 1 000 000).  
> В 2026 году:  
> - основной способ — `LAContext.evaluatePolicy(.deviceOwnerAuthentication, ...)`  
> - поддержка маски и очков — с 2022 года  
> - проверка доступности — `canEvaluatePolicy` + `biometryType == .faceID`  
> - обработка ошибок — **обязательна** (особенно `.biometryLockout`, `.userFallback`)  
> Это **стандарт де-факто** для входа, платежей и защиты данных в iOS-приложениях.

Удачи с безопасной и удобной аутентификацией в твоём приложении! 🔒👤