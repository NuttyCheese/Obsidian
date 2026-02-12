## 📘 Определение

**Fallback** — механизм или вариант резервного действия, используемый, когда основной метод не сработал.  
В [[iOS]] и [[Swift]] часто применяется вместе с **[[FaceID]] / [[TouchID]]**: если биометрическая аутентификация не удалась или недоступна, используется **резервный метод** (например, пароль или PIN-код).

---

## 🔹 Примеры кода

### 1. Простое использование fallback в [[LocalAuthentication]]

```swift
import LocalAuthentication

let context = LAContext()
context.localizedFallbackTitle = "Введите пароль" // текст кнопки fallback

context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "Авторизация") { success, error in
    DispatchQueue.main.async {
        if success {
            print("Аутентификация прошла")
        } else {
            print("Ошибка или fallback:", error?.localizedDescription ?? "")
        }
    }
}
```

---

### 2. Проверка, почему сработал fallback

```swift
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "Авторизация") { success, error in
    DispatchQueue.main.async {
        if success {
            print("Аутентификация через Face ID / Touch ID")
        } else if let laError = error as? LAError, laError.code == .userFallback {
            print("Пользователь выбрал fallback (например, пароль)")
        } else {
            print("Ошибка:", error?.localizedDescription ?? "")
        }
    }
}
```

---

### 3. Обработка fallback в UI

```swift
class ViewController: UIViewController {
    let context = LAContext()

    func authenticate() {
        context.localizedFallbackTitle = "Использовать пароль"
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "Доступ к данным") { success, error in
            DispatchQueue.main.async {
                if let laError = error as? LAError, laError.code == .userFallback {
                    self.showPasswordPrompt()
                }
            }
        }
    }

    func showPasswordPrompt() {
        print("Показываем UI для ввода пароля")
    }
}
```

---

### 4. Fallback на стандартную аутентификацию

```swift
context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: "Авторизация") { success, error in
    if success {
        print("Аутентификация успешна (биометрия или пароль)")
    } else {
        print("Ошибка:", error?.localizedDescription ?? "")
    }
}
```

---

### 5. Настройка fallback текста

```swift
let context = LAContext()
context.localizedFallbackTitle = "Использовать PIN"
```
