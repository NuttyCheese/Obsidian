#crash #fata_error #xcode #swift
### Что это значит

[[Xcode]] сообщает, что **линковка проекта завершилась с ошибкой**.

- Это **build-time ошибка**, связанная с **объединением скомпилированных модулей в итоговый исполняемый файл**.
    
- `exit code 1` — общее обозначение ошибки линковки.
    
- Часто Xcode не показывает подробности сразу, но можно использовать **Build with Verbose (`-v`)** для диагностики.
    

Основные причины:

1. **Undefined symbols** — вызов функции/метода, который не найден на этапе линковки.
    
2. **Duplicate symbols** — дублирование функций, классов или переменных.
    
3. **Missing framework/library** — зависимость не включена или отсутствует.
    
4. Несовместимые архитектуры (например, x86_64 vs arm64).
    
5. Ошибки при подключении [[Swift]] и [[Objective-C]] кода.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Undefined symbols**

```swift
// MyClass.swift
func doSomething() {}

// main.swift
doSomethingElse() // ⚠️ Undefined symbols for architecture x86_64
```

- Метод `doSomethingElse` не реализован → линковка падает.
    

---

**Пример 2: Duplicate symbols**

- Два файла содержат:
    

```swift
let sharedValue = 42
```

- Компилятор создаёт символ `sharedValue` дважды → линковщик выдаёт ошибку.
    

---

**Пример 3: Отсутствие фреймворка**

- Используется `Alamofire` в коде, но **не добавлен в Link Binary With Libraries** → линковка падает.
    

---

### Как исправить

#### 1️⃣ Проверить undefined symbols

- Чаще всего ошибка указывает, **какие символы не найдены**.
    
- Реализовать недостающие функции/классы или подключить нужные фреймворки.
    

---

#### 2️⃣ Проверить duplicate symbols

- Убедиться, что нет **дублирующихся функций, переменных, классов**.
    
- Для глобальных переменных использовать `static` или `private`:
    

```swift
private let sharedValue = 42
```

---

#### 3️⃣ Проверить Frameworks & Libraries

- **TARGETS → Build Phases → Link Binary With Libraries**
    
- Добавить все используемые зависимости.
    

---

#### 4️⃣ Проверить архитектуры

- **Build Settings → Architectures → Valid Architectures**
    
- Убедиться, что все библиотеки поддерживают нужную архитектуру.
    

---

#### 5️⃣ Verbose лог

- В Xcode: **Product → Build with Verbose Logging**
    
- Позволяет увидеть точную команду линковщика и понять причину ошибки.
    

---

### Резюме

- `Linker command failed with exit code 1` — это **общая ошибка линковщика**.
    
- Причины: **undefined symbols, duplicate symbols, missing frameworks, architecture mismatch**.
    
- Исправляется проверкой кода, зависимостей и архитектур.
    

---
