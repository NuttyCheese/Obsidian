**LocalAuthentication** — фреймворк Apple, предоставляющий интерфейс для **биометрической аутентификации** ([[FaceID]], [[TouchID]]) и альтернативной аутентификации устройством (пароль, PIN).  
Позволяет **защитить доступ к данным и функционалу приложения**, не храня пароли напрямую.  
Относится к **[[Foundation]] → Security / Authentication**.

---

## 🔹 Примеры кода

### 1. Проверка доступности биометрии

```swift
import LocalAuthentication

let context = LAContext()
var error: NSError?

if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    print("Биометрия доступна")
} else {
    print("Биометрия недоступна:", error?.localizedDescription ?? "")
}
```

---

### 2. Аутентификация с Face ID / Touch ID

```swift
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                       localizedReason: "Авторизуйтесь для доступа к секретам") { success, error in
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

### 3. Обработка [[fallback]] на пароль

```swift
context.localizedFallbackTitle = "Введите пароль"

context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: "Авторизация") { success, error in
    if success {
        print("Успешная аутентификация")
    } else {
        print("Ошибка:", error?.localizedDescription ?? "")
    }
}
```

---

### 4. Проверка типа биометрии

```swift
if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    switch context.biometryType {
    case .faceID:
        print("Используется Face ID")
    case .touchID:
        print("Используется Touch ID")
    default:
        print("Биометрия недоступна")
    }
}
```

---

### 5. Использование в функции для переиспользования

```swift
func authenticateUser(completion: @escaping (Bool) -> Void) {
    let context = LAContext()
    context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                           localizedReason: "Авторизация") { success, _ in
        DispatchQueue.main.async {
            completion(success)
        }
    }
}

authenticateUser { success in
    print(success ? "Доступ разрешен" : "Доступ запрещен")
}
```
