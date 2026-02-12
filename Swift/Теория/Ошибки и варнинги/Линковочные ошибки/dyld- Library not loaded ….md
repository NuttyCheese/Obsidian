#crash #fata_error #xcode #swift
### Что это значит

Ошибка появляется при **запуске приложения**, когда **динамическая библиотека (dylib/framework) не может быть найдена или загружена**.

- Это **runtime ошибка**, связанная с **динамической линковкой**.
    
- Обычно вывод в консоли выглядит так:
    

```
dyld: Library not loaded: @rpath/SomeFramework.framework/SomeFramework
  Referenced from: /path/to/app
  Reason: image not found
```

Причины:

1. Библиотека **не включена в bundle** приложения.
    
2. Ошибки в настройках **Runpath Search Paths** (`@rpath`).
    
3. Неправильный **Build Phase → Embed Frameworks**.
    
4. Сборка для одного таргета, а запуск пытается использовать другой (например, тестовый таргет).
    
5. Проблемы с **[[Swift]] version mismatch** между приложением и библиотекой.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Использование стороннего фреймворка без embed**

- Подключаем `Alamofire.framework` через [[SPM]] или manual, но не добавили в **Embed Frameworks**.
    
- При запуске:
    

```
dyld: Library not loaded: @rpath/Alamofire.framework/Alamofire
```

---

**Пример 2: Ручная сборка с [[Carthage]]**

- Если `.framework` не скопирован в папку `Frameworks` приложения.
    

---

### Как исправить

#### 1️⃣ Embed Framework в [[Xcode]]

- **TARGETS → Build Phases → Embed Frameworks** → добавить missing framework.
    
- Убедиться, что **"Embed & Sign"** стоит для [[iOS]] приложений.
    

---

#### 2️⃣ Проверить Runpath Search Paths

- **TARGETS → Build Settings → Runpath Search Paths** (`@executable_path/Frameworks`)
    
- Обычно для iOS:
    

```
@executable_path/Frameworks
```

- Для macOS:
    

```
@loader_path/../Frameworks
```

---

#### 3️⃣ Проверить Bundle

- Все `.framework` должны быть в **Copy Bundle Resources / Frameworks**.
    

---

#### 4️⃣ Сборка и чистка

```bash
Cmd+Shift+K  // Clean Build Folder
Cmd+B       // Rebuild
```

- Иногда ошибка связана с кешем старой сборки.
    

---

#### 5️⃣ Для SPM

- Проверить, что пакет **указан как dependency** и **включен в target**.
    

---

### Резюме

- `dyld: Library not loaded` — это **runtime линковочная ошибка**, указывающая на отсутствие динамической библиотеки в приложении.
    
- Исправляется через:
    
    1. Добавление в Embed Frameworks.
        
    2. Проверку `Runpath Search Paths`.
        
    3. Чистую сборку приложения.
        
    4. Проверку зависимостей [[SPM]]/[[Carthage]].
        

---
