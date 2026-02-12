#crash #warning #xcode #swift
### Что это значит

Это предупреждение возникает, когда [[Xcode]] игнорирует указанные **Runpath Search Paths** (`@rpath`) при линковке.

Причины:

1. Целевая библиотека или framework не существует по указанному пути.
    
2. Framework добавлен, но не включён в **Embed Frameworks**.
    
3. Используется неправильная конфигурация для **Dynamic Library** (`.dylib` / `.framework`).
    
4. Конфликт версий библиотек или настройка пути (`LD_RUNPATH_SEARCH_PATHS`) некорректна.
    

Это не всегда блокирует сборку, но может привести к **runtime crash**, если приложение не сможет загрузить динамическую библиотеку.

---

### Примеры кода/сценариев возникновения

**Пример 1: Dynamic Framework**

- В `Build Settings` указано:
    

```
LD_RUNPATH_SEARCH_PATHS = @executable_path/Frameworks
```

- Framework `SomeFramework.framework` не добавлен в **Embed Frameworks** → предупреждение:
    

```
Runpath search paths were ignored because … SomeFramework.framework does not exist
```

---

**Пример 2: [[CocoaPods]] / [[SPM]]**

- Подключена библиотека через SPM, но Runpath не обновился → предупреждение появляется при сборке симулятора или устройства.
    

---

### Как исправить

#### 1️⃣ Проверка Embed Frameworks

- Target → General → **Frameworks, Libraries, and Embedded Content**
    
- Убедитесь, что динамические фреймворки отмечены как **Embed & Sign**.
    

#### 2️⃣ Проверка пути Runpath

- Target → Build Settings → **Runpath Search Paths (`LD_RUNPATH_SEARCH_PATHS`)**
    
- Часто достаточно:
    

```
@executable_path/Frameworks
@loader_path/Frameworks
```

#### 3️⃣ Проверка наличия библиотек

- Убедитесь, что все динамические `.framework` файлы действительно находятся в каталоге проекта и включены в сборку.
    

#### 4️⃣ Очистка проекта

- Xcode → Product → Clean Build Folder (`Shift + Cmd + K`)
    
- Иногда старые пути остаются в кэше.
    

---

### Пример исправления

**Было:**

- Framework добавлен в проект, но не в **Embed Frameworks** → предупреждение:
    

```
Runpath search paths were ignored because SomeFramework.framework does not exist
```

**Исправлено:**

1. Target → General → **Embed & Sign** для `SomeFramework.framework`.
    
2. LD_RUNPATH_SEARCH_PATHS оставлено как `@executable_path/Frameworks`.
    

Теперь Xcode корректно находит framework при запуске и предупреждение исчезает.

---
