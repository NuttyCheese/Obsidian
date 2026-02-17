[[Xcode]] не смог найти или подключить библиотеку, которая должна была автоматически линковаться (auto-linked).

Причины:

1. Библиотека отсутствует на диске или в проекте.
    
2. Неправильно настроены **Search Paths** (`Library Search Paths` / `Framework Search Paths`).
    
3. Конфликт версий библиотек или их архитектур (например, пытаемся линковать x86_64 и arm64 одновременно).
    
4. Используется модуль из Swift Package Manager, CocoaPods или Carthage, который не был правильно подключён к целевой платформе.
    
5. Проблемы с Build Phases → Link Binary With Libraries.
    

---

### Примеры кода возникновения

**Пример 1: использование библиотеки без подключения**

```swift
import Alamofire

AF.request("https://example.com").response { response in
    print(response)
}
```

Если библиотека Alamofire не добавлена через Swift Package Manager / CocoaPods / Carthage → Xcode может выдать:

```
Could not find or use auto-linked library 'Alamofire'
```

---

**Пример 2: попытка линковки framework, отсутствующего на устройстве**

В `Link Binary With Libraries` добавлен фреймворк `SomeFramework.framework`, но его нет на диске → при сборке появится предупреждение:

```
Could not find or use auto-linked library 'SomeFramework'
```

---

### Как исправить

#### 1️⃣ Проверка зависимостей

- Убедитесь, что библиотека установлена через **[[SPM]], [[CocoaPods]] или [[Carthage]]**.
    
- В SPM: File → Swift Packages → Update to Latest Package Versions.
    

#### 2️⃣ Проверка Search Paths

- В Target → Build Settings:
    
    - `Library Search Paths` → путь к библиотеке
        
    - `Framework Search Paths` → путь к фреймворку
        
- Используйте `$(PROJECT_DIR)/Pods/**` или путь к локальной библиотеке.
    

#### 3️⃣ Link Binary With Libraries

- Target → Build Phases → Link Binary With Libraries
    
- Добавьте нужный `.framework` или `.a` файл.
    

#### 4️⃣ Проверка архитектур

- Target → Build Settings → `Excluded Architectures`
    
- Убедитесь, что архитектура вашего устройства/симулятора поддерживается библиотекой.
    

---

### Пример исправления

**Используем Alamofire через SPM:**

1. File → Swift Packages → Add Package Dependency
    
2. Ввести: `https://github.com/Alamofire/Alamofire.git`
    
3. Подключить к нужному таргету
    

Код после исправления:

```swift
import Alamofire

AF.request("https://example.com").response { response in
    print(response)
}
```

Теперь предупреждение исчезает, потому что библиотека корректно подключена и auto-linked работает.

---
