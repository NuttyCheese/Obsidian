[[Xcode]] сообщает, что **процесс подписи приложения (Code Signing) завершился с ошибкой**.

- В [[iOS]]/macOS все приложения должны быть подписаны сертификатом разработчика для запуска на устройстве или публикации.
    
- Ошибка `Command CodeSign failed with a nonzero exit code` говорит о том, что **Xcode не смог корректно подписать приложение**.
    
- Часто встречается при сборке на реальном устройстве, архивации для App Store или при использовании сторонних библиотек.
    

---

### Причины возникновения

1. **Проблемы с сертификатами и профилями**
    
    - Истёкший сертификат или provisioning profile.
        
    - Несовпадение bundle identifier с profile.
        
2. **Неверные настройки Code Signing в [[Xcode]]**
    
    - В Project/Target → Signing & Capabilities: выбран неправильный team или profile.
        
3. **Повреждённые или отсутствующие ключи в Keychain**
    
    - Сертификаты разработчика не установлены или недоступны для Xcode.
        
4. **Сторонние фреймворки без корректной подписи**
    
    - Подпись библиотек, которые не подписаны, может вызвать ошибку.
        
5. **Очистка Derived Data/кэша Xcode**
    
    - Иногда старые кэши или конфликтующие подписи вызывают проблему.
        

---

### Примеры

**Пример 1: Истёкший сертификат**

- В Xcode видно, что сертификат «[[iOS]] Development» красным → CodeSign не может пройти.
    

**Пример 2: Несовпадение bundle identifier**

- В проекте: `com.example.myapp`
    
- В provisioning profile: `com.example.otherapp` → CodeSign ошибка.
    

**Пример 3: Подпись стороннего фреймворка**

```bash
Command CodeSign failed with a nonzero exit code
Signing Identity: "iPhone Developer: John Doe (ABCDE12345)"
```

- Фреймворк не подписан или не совпадает с идентификатором.
    

---

### Как исправить

#### 1️⃣ Проверка сертификатов и профилей

- Xcode → Preferences → Accounts → проверить сертификаты.
    
- Обновить или заново загрузить provisioning profile.
    
- Убедиться, что bundle identifier совпадает с profile.
    

#### 2️⃣ Настройки Code Signing в Target

- Xcode → Target → Signing & Capabilities
    
    - Team: выбрать правильную команду
        
    - Provisioning Profile: Automatic или правильный вручную
        
    - Signing Certificate: iOS Developer / Distribution
        

#### 3️⃣ Очистка Derived Data и кэшей

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData
```

- Перезапустить Xcode и пересобрать проект.
    

#### 4️⃣ Проверка сторонних библиотек

- Если используется [[CocoaPods]]/[[SPM]], убедиться, что фреймворки корректно подписаны.
    
- Иногда помогает повторная интеграция или включение `Code Sign On Copy` для фреймворков.
    

#### 5️⃣ Принудительная подпись через команду

```bash
codesign --force --sign "iPhone Developer: John Doe (ABCDE12345)" MyApp.app
```

- Используется для диагностики или исправления локально.
    

---

### Резюме

- Ошибка возникает, когда **Xcode не может подписать приложение**.
    
- Исправляется через:
    
    1. Проверку сертификатов и provisioning profiles.
        
    2. Настройку Target → Signing & Capabilities.
        
    3. Очистку Derived Data и кэша.
        
    4. Проверку и подпись сторонних библиотек.
        
- Позволяет успешно собрать и запустить приложение на устройстве или отправить в App Store.
    

---
