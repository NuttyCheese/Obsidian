[[Xcode]] сообщает, что **для подписи приложения требуется выбрать команду разработчиков (Development Team)**.

- Ошибка возникает, когда:
    
    1. **Не выбран Team** в настройках подписи.
        
    2. **Автоматическая подпись включена**, но Xcode не знает, какой Team использовать.
        
    3. Проект использует **новый Bundle Identifier**, и профиля для него ещё нет.
        
- Без указания команды Xcode **не может подписать приложение** для сборки на устройстве или App Store.
    

---

### Примеры сценариев возникновения

**Пример 1: Новый проект**

- Создан новый проект в Xcode
    
- Target → Signing & Capabilities → Team: None
    
- Попытка сборки на устройстве → ❌ Signing for "MyApp" requires a development team
    

---

**Пример 2: Автоматическая подпись без выбора Team**

- Включена **Automatically manage signing**
    
- Team не выбран → ошибка при сборке
    

---

**Пример 3: Смена Bundle Identifier**

- Сменили Bundle Identifier на `com.example.myapp`
    
- Профиля для этого идентификатора нет, а Team не выбран → ошибка
    

---

### Как исправить

#### 1️⃣ Выбрать команду разработчиков (Team)

- Target → Signing & Capabilities → Team → выбрать ваш Apple Developer Team
    
- После выбора Xcode может автоматически создать provisioning profile
    

---

#### 2️⃣ Включить автоматическую подпись (если возможно)

- Отметить **Automatically manage signing**
    
- Xcode сгенерирует подходящий профиль для выбранного Team
    

---

#### 3️⃣ Проверить Bundle Identifier

- Target → General → Identity → Bundle Identifier
    
- Должен соответствовать профилю Team или автоматически сгенерированному профилю
    

---

#### 4️⃣ Если нет аккаунта разработчика

- Нужно зарегистрироваться в [Apple Developer Program](https://developer.apple.com/programs/)
    
- Добавить аккаунт в Xcode → Preferences → Accounts
    

---

### Резюме

- **Signing for … requires a development team** появляется, когда Xcode **не знает, какой Team использовать для подписи приложения**.
    
- Исправляется через:
    
    1. Выбор Team в Target → Signing & Capabilities
        
    2. Включение автоматической подписи
        
    3. Проверку Bundle Identifier
        
    4. Добавление Apple Developer аккаунта, если он отсутствует
        
- Позволяет Xcode подписывать приложение для устройства и App Store.