**Face ID** — технология распознавания лица от Apple для аутентификации пользователя.  
Позволяет безопасно разблокировать устройство, подтверждать покупки и входить в приложения.  
В [[iOS]] доступна через фреймворк **[[LocalAuthentication]]** (`LAContext`).

---

## 🔹 Примеры кода

### 1. Простая проверка поддержки Face ID

```swift
import LocalAuthentication

let context = LAContext()
var error: NSError?

if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    if context.biometryType == .faceID {
        print("Face ID доступен")
    } else {
        print("Доступна другая биометрия")
    }
} else {
    print("Биометрия недоступна:", error?.localizedDescription ?? "")
}
```

---

### 2. Аутентификация с Face ID

```swift
let context = LAContext()
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "Авторизация через Face ID") { success, error in
    DispatchQueue.main.async {
        if success {
            print("Аутентификация прошла успешно")
        } else {
            print("Ошибка аутентификации:", error?.localizedDescription ?? "")
        }
    }
}
```

---

### 3. Обработка ошибок Face ID

```swift
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "Доступ к секретам") { success, error in
    DispatchQueue.main.async {
        if let laError = error as? LAError {
            switch laError.code {
            case .authenticationFailed:
                print("Не удалось распознать лицо")
            case .userCancel:
                print("Пользователь отменил")
            case .userFallback:
                print("Пользователь выбрал пароль")
            case .biometryNotAvailable:
                print("Face ID недоступен")
            default:
                print("Другая ошибка:", laError.localizedDescription)
            }
        }
    }
}
```

---

### 4. Face ID с [[fallback]] на пароль

```swift
context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: "Авторизация") { success, error in
    DispatchQueue.main.async {
        if success {
            print("Аутентификация прошла")
        } else {
            print("Ошибка:", error?.localizedDescription ?? "")
        }
    }
}
```

---

### 5. Проверка биометрии перед Face ID

```swift
if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    switch context.biometryType {
    case .faceID:
        print("Используем Face ID")
    case .touchID:
        print("Используем Touch ID")
    default:
        print("Биометрия не поддерживается")
    }
} else {
    print("Биометрия недоступна")
}
```
