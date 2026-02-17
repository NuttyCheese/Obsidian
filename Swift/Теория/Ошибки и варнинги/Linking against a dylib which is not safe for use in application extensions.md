Это предупреждение появляется, когда вы **линкуете динамическую библиотеку (`.dylib` или `.framework`)**, которая **не помечена как безопасная для использования в App Extensions**.

- App Extension (например, Widget, Notification Content Extension, Siri Shortcut) имеет ограничения на доступ к [[API]] [[iOS]].
    
- Если ваше расширение использует небезопасную библиотеку, это может привести к **runtime crash** или отклонению приложения на App Store.
    

Причины появления:

1. Использование обычного фреймворка, который не отмечен с `APPLICATION_EXTENSION_API_ONLY = YES`.
    
2. Линковка сторонних библиотек, которые не поддерживают App Extensions.
    
3. Попытка использовать API, запрещённые в расширениях (например, `UIApplication.shared`).
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Widget использует [[UIKit]] API напрямую**

```swift
// В NotificationContentExtension
import UIKit

let app = UIApplication.shared // ⚠️ Linking against a dylib which is not safe for use in application extensions
```

- UIKit частично безопасен, но `UIApplication.shared` запрещён → предупреждение компилятора.
    

---

**Пример 2: Линковка стороннего фреймворка**

- В Build Phases → Link Binary With Libraries добавлен `SomeFramework.framework`.
    
- Этот framework не поддерживает App Extensions → предупреждение:
    

```
Linking against a dylib which is not safe for use in application extensions
```

---

### Как исправить

#### 1️⃣ Проверка `APPLICATION_EXTENSION_API_ONLY`

- Target → Build Settings → **Require Only App-Extension-Safe API = YES**
    
- Убедитесь, что все линкованные библиотеки помечены как безопасные.
    

#### 2️⃣ Использовать App Extension–совместимые версии библиотек

- Проверьте документацию сторонней библиотеки.
    
- Для [[CocoaPods]]: `pod 'SomeFramework', :modular_headers => true` иногда помогает.
    

#### 3️⃣ Избегать запрещённых API

- В App Extension нельзя использовать:
    
    - `UIApplication.shared`
        
    - `SharedApplication`
        
    - `openURL(_:)`
        
    - Некоторые функции [[UIKit]], [[Core Location]], [[AVFoundation]] и т.д.
        
- Используйте безопасные альтернативы или передавайте данные через контейнер App Group.
    

#### 4️⃣ Создать отдельную библиотеку для расширений

- Если используется свой фреймворк, соберите его с **APPLICATION_EXTENSION_API_ONLY = YES**.
    

---

### Пример исправления

**Было:**

```swift
let app = UIApplication.shared // ⚠️ запрещён в расширении
```

**Исправлено:**

```swift
// Для передачи данных используем UserDefaults с App Group
let defaults = UserDefaults(suiteName: "group.com.example.app")
defaults?.set("value", forKey: "key")
```

- Все API безопасны → предупреждение исчезает.
    

---
