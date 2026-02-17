[[Xcode]] предупреждает, что **целевой [[iOS]] Deployment Target** вашего таргета установлен вне допустимого диапазона версий SDK.

- **Deployment Target** — минимальная версия iOS, на которой приложение может работать.
    
- Если указана слишком старая или новая версия по сравнению с установленным SDK, Xcode выдаёт предупреждение.
    
- Причины:
    
    1. Используется устаревший Deployment Target, который больше не поддерживается новым SDK.
        
    2. Deployment Target установлен выше текущего SDK → несовместимость.
        
    3. Подключён сторонний фреймворк, у которого минимальная поддерживаемая версия выше вашего Deployment Target.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Старый Deployment Target**

- Deployment Target проекта: **iOS 9.0**
    
- Используется Xcode 15 → минимальная поддерживаемая версия iOS 11.0
    

Предупреждение:

```
The iOS deployment target '9.0' is set for the target 'MyApp', but the range of supported deployment target versions is '11.0–17.0'.
```

---

**Пример 2: Несовместимый сторонний фреймворк**

- Deployment Target проекта: **iOS 13.0**
    
- Подключён framework, минимальная поддержка iOS 14.0
    

Предупреждение:

```
The iOS deployment target '13.0' is set for the target 'MyApp', but 'SomeFramework' requires a minimum deployment target of '14.0'.
```

---

### Как исправить

#### 1️⃣ Обновить Deployment Target проекта/таргета

- Target → Build Settings → **iOS Deployment Target**
    
- Установить минимально поддерживаемую версию, совместимую с SDK и фреймворками.
    

#### 2️⃣ Проверить сторонние библиотеки

- Убедиться, что все фреймворки поддерживают выбранный Deployment Target.
    
- Если фреймворк требует более новую версию iOS → либо обновить проект, либо заменить библиотеку.
    

#### 3️⃣ Проверить настройки Pods (если используется [[CocoaPods]])

- В Podfile можно указать platform:
    

```ruby
platform :ios, '14.0'
```

- Затем выполнить `pod install` для синхронизации.
    

---

### Пример исправления

**Было:**

- Deployment Target = iOS 9.0
    
- Xcode 15 поддерживает только iOS 11+
    
- Предупреждение при сборке.
    

**Исправлено:**

- Target → Build Settings → iOS Deployment Target = **iOS 11.0**
    
- Все библиотеки и фреймворки поддерживают iOS 11+
    
- Предупреждение исчезает, сборка успешна.
    

---
