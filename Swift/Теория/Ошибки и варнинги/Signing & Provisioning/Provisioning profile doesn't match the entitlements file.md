#crash #fata_error #xcode #swift
### Что это значит

[[Xcode]] сообщает, что **выбранный provisioning profile не соответствует текущему файлу entitlements** проекта.

- **Entitlements** — это специальные права (capabilities) приложения:
    
    - Push Notifications
        
    - App Groups
        
    - Keychain Access
        
    - iCloud и т.д.
        
- Ошибка возникает, когда:
    
    1. **Provisioning profile не включает требуемые права**.
        
    2. Bundle Identifier профиля и проекта совпадают, но профайл не настроен на нужные capabilities.
        
    3. Файл `.entitlements` был изменён после создания профиля.
        
- Xcode не может подписать приложение, если профайл **не поддерживает все включенные возможности**.
    

---

### Примеры сценариев возникновения

**Пример 1: Push Notifications**

- Включены Push Notifications в проекте (Target → Signing & Capabilities → Push Notifications).
    
- Используется provisioning profile без включённого Push Notifications → ошибка.
    

---

**Пример 2: App Groups**

- Добавлена capability App Groups.
    
- Профиль был создан раньше, без этой capability → конфликт с entitlements → ошибка.
    

---

**Пример 3: iCloud**

- В проекте включён iCloud.
    
- Профиль не поддерживает iCloud → Xcode выдаёт ошибку при сборке.
    

---

### Как исправить

#### 1️⃣ Обновить Provisioning Profile

- Перейти на [Apple Developer Portal](https://developer.apple.com/account/resources/profiles/list)
    
- Найти профиль для вашего Bundle Identifier
    
- Убедиться, что **включены все необходимые capabilities**
    
- Обновить профиль → скачать → установить в Xcode
    

---

#### 2️⃣ Включить/отключить capabilities в проекте

- Target → Signing & Capabilities
    
- Проверить, что все capabilities **соответствуют профилю**
    
- Если профайл не поддерживает capability — временно отключить её
    

---

#### 3️⃣ Автоматическая подпись (если возможно)

- Включить **Automatically manage signing**
    
- Xcode попытается сгенерировать профиль с нужными capabilities
    

---

#### 4️⃣ Проверить Bundle Identifier

- Убедиться, что **Bundle Identifier проекта совпадает с профилем**
    
- Target → General → Identity → Bundle Identifier
    

---

### Резюме

- **Provisioning profile doesn't match the entitlements file** возникает, когда **профиль не поддерживает все возможности, указанные в entitlements проекта**.
    
- Исправляется через:
    
    1. Обновление профиля с нужными capabilities
        
    2. Проверку и корректировку capabilities проекта
        
    3. Использование автоматической подписи Xcode
        
    4. Проверку Bundle Identifier
        
- Позволяет Xcode корректно подписывать приложение и использовать все включённые возможности.