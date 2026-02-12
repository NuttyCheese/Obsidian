#crash #fata_error #xcode #swift
### Что это значит

[[Xcode]] сообщает, что **не удалось найти подходящий provisioning profile** для сборки или запуска приложения на устройстве или симуляторе.

- Ошибка возникает при сборке проекта под [[iOS]] или Mac, когда:
    
    1. **Provisioning Profile отсутствует** в системе.
        
    2. **Bundle Identifier не совпадает** с профилем.
        
    3. **Сертификаты разработчика истекли или отсутствуют**.
        
    4. Автоматическая подпись не может подобрать профиль.
        
- Обычно сопровождается сообщением:
    
    ```
    No profiles for 'com.example.app' were found: Xcode couldn't find any iOS App Development provisioning profiles matching 'com.example.app'.
    ```
    

---

### Примеры сценариев возникновения

**Пример 1: Новый проект без профиля**

- Создан новый Bundle Identifier `com.example.myapp`.
    
- В Xcode включена автоматическая подпись, но профиль ещё не сгенерирован → ошибка.
    

---

**Пример 2: Несовпадение Bundle Identifier**

- Профиль создан для `com.example.app`, а проект использует `com.example.myapp`.
    
- Xcode не может найти подходящий профиль → ошибка.
    

---

**Пример 3: Истёкший сертификат**

- Provisioning profile был действителен, но срок сертификата разработчика истёк.
    
- Профиль недействителен → ошибка при сборке.
    

---

### Как исправить

#### 1️⃣ Включить автоматическую подпись (Automatic Signing)

- В Xcode → Target → **Signing & Capabilities**
    
- Отметить **Automatically manage signing**
    
- Выбрать свой **Team**
    
- Xcode сгенерирует подходящий provisioning profile автоматически
    

---

#### 2️⃣ Проверить Bundle Identifier

- Убедиться, что **Bundle Identifier проекта совпадает** с профилем.
    
- Исправить при необходимости в Target → General → Identity → Bundle Identifier
    

---

#### 3️⃣ Обновить или создать профиль вручную

- В Apple Developer → Certificates, IDs & Profiles → Provisioning Profiles
    
- Создать **[[iOS]] App Development profile** для вашего Bundle Identifier
    
- Скачать и установить профиль в Xcode
    

---

#### 4️⃣ Проверить сертификаты разработчика

- Xcode → Preferences → Accounts → Manage Certificates
    
- Обновить или создать новый Development certificate
    

---

### Резюме

- **No profiles for … were found** означает, что Xcode **не смог найти подходящий provisioning profile для подписи приложения**.
    
- Исправляется через:
    
    1. Автоматическую подпись и выбор Team
        
    2. Проверку и исправление Bundle Identifier
        
    3. Создание или обновление provisioning profile вручную
        
    4. Проверку и обновление сертификатов разработчика
        
- Позволяет Xcode корректно подписывать приложение и собирать его на устройстве или для App Store.