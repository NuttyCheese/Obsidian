#system_design
## Определение

**Биометрическая аутентификация** — это способ подтверждения личности пользователя с помощью **уникальных физических характеристик**:

- **FaceID** → распознавание лица.
    
- **TouchID** → распознавание отпечатка пальца.
    

Цель: **повысить безопасность приложений и упростить вход**, снижая необходимость ввода пароля.

---

## 1. Основные возможности

- **Аутентификация для входа в приложение**.
    
- **Подтверждение транзакций** (например, платежей).
    
- **Доступ к защищённым данным** (например, сохранённые пароли).
    
- **Интеграция с Keychain** для безопасного хранения токенов.
    

---

## 2. Принципы работы

1. **Регистрация биометрии на устройстве**
    
    - [[TouchID]] / [[FaceID]] конфигурируется в настройках [[iOS]].
        
    - Биометрические данные хранятся **только на устройстве** в Secure Enclave.
        
2. **Проверка биометрии в приложении**
    
    - Приложение запрашивает проверку через **LocalAuthentication framework**.
        
    - Система сравнивает введённые данные с хранящимися шаблонами.
        
    - Приложение получает только **успех или отказ**, без доступа к самим биометрическим данным.
        

---

## 3. Интеграция в iOS ([[Swift]])

```swift
import LocalAuthentication

func authenticateUser(completion: @escaping (Bool, Error?) -> Void) {
    let context = LAContext()
    var error: NSError?

    // Проверка доступности биометрии
    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
        let reason = "Authenticate to access secure data"

        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: reason) { success, authError in
            DispatchQueue.main.async {
                completion(success, authError)
            }
        }
    } else {
        // Биометрия недоступна, fallback на пароль
        completion(false, error)
    }
}
```

### Особенности

- **canEvaluatePolicy** → проверка доступности TouchID / FaceID на устройстве.
    
- **evaluatePolicy** → запрос биометрической аутентификации.
    
- Обработчик возвращает **успех или ошибку**.
    

---

## 4. Типы ошибок

- **LAError.authenticationFailed** → пользователь не прошёл проверку.
    
- **LAError.userCancel** → пользователь отменил аутентификацию.
    
- **LAError.userFallback** → пользователь выбрал альтернативный способ (например, пароль).
    
- **LAError.biometryNotAvailable** → устройство не поддерживает биометрию.
    
- **LAError.biometryLockout** → слишком много неудачных попыток.
    

---

## 5. Интеграция с Keychain

- Биометрия часто используется совместно с **[[Keychain]]**, чтобы безопасно хранить токены или пароли.
    
- Пример хранения токена с защитой биометрией:
    

```swift
let access = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryCurrentSet,
    nil
)!

let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "userToken",
    kSecValueData as String: tokenData,
    kSecAttrAccessControl as String: access
]

SecItemAdd(query as CFDictionary, nil)
```

- Доступ к токену требует успешной биометрической аутентификации.
    

---

## 6. Best Practices

1. **Fallback на пароль или PIN** → для устройств без биометрии или при сбое.
    
2. **Использовать Secure Enclave и [[Keychain]]** → хранение чувствительных данных только там.
    
3. **Обрабатывать все ошибки** → информировать пользователя корректно.
    
4. **Не хранить биометрические данные в приложении** → это делает iOS автоматически.
    
5. **Обновлять UI при блокировке/разблокировке** → избегать попыток аутентификации в фоне.
    

---

## 7. Применение в мобильных приложениях

- **Финансовые приложения** → подтверждение платежей и вход.
    
- **Мессенджеры и социальные сети** → быстрый вход без пароля.
    
- **Приложения с личными данными** → доступ к защищённой информации.
    

---

## Итог

- **FaceID и TouchID** → быстрый и безопасный способ аутентификации.
    
- **LocalAuthentication framework** → основной инструмент для интеграции в iOS.
    
- **Keychain + биометрия** → безопасное хранение токенов и паролей.
    
- Best Practices: fallback, обработка ошибок, Secure Enclave, минимизация UX friction.
    

---
