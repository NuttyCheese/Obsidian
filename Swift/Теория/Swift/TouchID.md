## 📘 Определение

**Touch ID** — биометрическая технология Apple для **идентификации пользователя по отпечатку пальца**.  
Используется для **разблокировки устройств, авторизации в приложениях, подтверждения платежей**.  
В [[iOS]] доступна через **фреймворк [[LocalAuthentication]]** (`LAContext`).  
Относится к **iOS → Security / Authentication**.

---

## 🔹 Примеры кода

### 1. Проверка доступности Touch ID

```swift
import LocalAuthentication

let context = LAContext()
var error: NSError?

if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    print("Touch ID доступен")
} else {
    print("Touch ID недоступен:", error?.localizedDescription ?? "неизвестная причина")
}
```

---

### 2. Аутентификация с Touch ID

```swift
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                       localizedReason: "Авторизуйтесь с помощью Touch ID") { success, error in
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

### 3. Обработка наличия [[FaceID]] и TouchID

```swift
switch context.biometryType {
case .none:
    print("Биометрия не доступна")
case .touchID:
    print("Используется Touch ID")
case .faceID:
    print("Используется Face ID")
@unknown default:
    print("Неизвестный тип биометрии")
}
```

---

### 4. Использование с [[fallback]] паролем

```swift
context.evaluatePolicy(.deviceOwnerAuthentication,
                       localizedReason: "Авторизуйтесь с помощью Touch ID или пароля") { success, error in
    if success {
        print("Авторизация успешна")
    } else {
        print("Ошибка:", error?.localizedDescription ?? "")
    }
}
```

---

### 5. Асинхронная обработка с [[Combine]]

```swift
import Combine

func authenticateWithTouchID() -> Future<Bool, Error> {
    return Future { promise in
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                               localizedReason: "Авторизация") { success, error in
            if let error = error {
                promise(.failure(error))
            } else {
                promise(.success(success))
            }
        }
    }
}
```
