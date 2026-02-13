### 1. Что такое Use After Free

**Use After Free** (UAF) — это **критическая ошибка управления памятью**, при которой программа **обращается к объекту или области памяти после того, как она была освобождена** (deallocated).

В [[Swift]] такое случается **гораздо реже**, чем в C/C++/[[Objective-C]], благодаря **[[ARC]]** (Automatic Reference Counting), но **всё равно возможно** в следующих случаях:

- Использование `unowned` ссылок, когда объект уже уничтожен  
- Работа с **Unsafe**-указателями (`UnsafePointer`, `UnsafeMutablePointer`, `UnsafeRawPointer`)  
- Взаимодействие с **C API** или низкоуровневыми фреймворками ([[Core Foundation]], [[Metal]], [[Core Audio]], [[Accelerate]] и т.д.)  
- Неправильное использование `Unmanaged` или ручное управление retain/release  
- Ошибки в **unsafe bit casting** или **memory layout**  
- Редко — баги в ARC или компиляторе (крайне редко в 2026 году)

**Самые частые последствия UAF в iOS-приложении**:

- `EXC_BAD_ACCESS` (KERN_INVALID_ADDRESS) — классический краш  
- `SIGSEGV` / `SIGBUS`  
- Неопределённое поведение (undefined behavior): мусорные данные, неправильная логика  
- Случайные краши только на реальных устройствах (симулятор может «проглатывать»)  
- Очень трудно воспроизводимые баги («иногда крашится, иногда нет»)

### 2. Самые частые сценарии Use After Free в Swift 2026

| №  | Сценарий                                      | Как проявляется                              | Частота |
|----|-----------------------------------------------|----------------------------------------------|---------|
| 1  | `unowned` ссылка после deinit владельца       | Краш при обращении к `unowned` свойству      | ★★★★★   |
| 2  | UnsafeMutablePointer после deallocate()       | EXC_BAD_ACCESS при pointee / pointee =       | ★★★★☆   |
| 3  | Core Foundation объект после CFRelease        | Краш при вызове метода CF-объекта            | ★★★★☆   |
| 4  | Metal buffer / texture после release          | GPU краш или артефакты на экране             | ★★★☆☆   |
| 5  | Accelerate / vDSP буфер после free            | Мусорные результаты вычислений               | ★★★☆☆   |
| 6  | Unmanaged.passUnretained после release        | Краш при использовании объекта               | ★★★☆☆   |
| 7  | Ошибка в unsafe bit cast / memory layout      | Неопределённое поведение, краш               | ★★☆☆☆   |

### 3. Классические примеры Use After Free (и как они крашатся)

#### Пример 1 — Самый частый: unowned после deinit

```swift
class Owner {
    var child: Child?
    deinit { print("Owner deinit") }
}

class Child {
    unowned let owner: Owner  // ← опасно!
    
    init(owner: Owner) {
        self.owner = owner
    }
    
    deinit { print("Child deinit") }
    
    func sayHello() {
        print("Hello from \(owner)")  // ← краш, если owner уже deinit
    }
}

var owner: Owner? = Owner()
owner?.child = Child(owner: owner!)

owner = nil  // Owner и Child deinit

owner?.child?.sayHello()  // ← EXC_BAD_ACCESS! Use After Free
```

**Почему краш**: `unowned` не увеличивает retain count → когда `owner` освобождается, ссылка в `child` становится dangling pointer.

#### Пример 2 — UnsafeMutablePointer после deallocate

```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
ptr.initialize(to: 42)

print(ptr.pointee)  // 42 — ок

ptr.deallocate()    // память освобождена

print(ptr.pointee)  // ← Use After Free! EXC_BAD_ACCESS или мусор
```

**Правильно**:

```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
defer { ptr.deallocate() }  // defer спасает

ptr.initialize(to: 42)
print(ptr.pointee)  // безопасно
```

#### Пример 3 — Core Foundation после CFRelease

```swift
var cfString: CFString? = "Hello" as CFString
CFRelease(cfString!)  // память освобождена

CFStringGetLength(cfString!)  // ← Use After Free! Краш или UB
```

**Правильно**: используйте `Unmanaged` или ARC-объекты:

```swift
let nsString = "Hello" as NSString  // ARC управляет
print(nsString.length)  // безопасно
```

### 4. Все современные способы защиты от Use After Free (2026)

| Способ защиты                     | Защита от UAF | Сложность | Рекомендация 2026 | Примечание |
|-----------------------------------|----------------|-----------|-------------------|------------|
| **weak вместо unowned**           | ★★★★★          | ★★☆☆☆     | Всегда            | Стандарт для ссылок на owner |
| **ARC + автоматическое управление** | ★★★★★        | ★☆☆☆☆     | Всегда            | Главный защитник в Swift |
| **defer { release / deallocate }** | ★★★★★       | ★★☆☆☆     | При Unsafe        | Спасает от забытого освобождения |
| **actor**                         | ★★★★★          | ★★★☆☆     | Для состояния     | Нет прямого доступа → нет UAF |
| **Sendable + strict concurrency** | ★★★★★          | ★★★★☆     | Swift 6 проекты   | Ловит dangling pointer на этапе компиляции |
| **Unsafe API с осторожностью**    | ★★★★☆          | ★★★★☆     | Только при необходимости | Использовать только когда нет альтернативы |
| **Unmanaged.passRetained / takeRetained** | ★★★★☆   | ★★★★☆     | CF-объекты        | Контроль retain/release |

### 5. Как найти Use After Free в 2026

| Инструмент / Способ               | Что ловит                              | Где включается                  | Эффективность |
|-----------------------------------|----------------------------------------|----------------------------------|---------------|
| **Address Sanitizer**             | Use After Free, Use After Scope, buffer overflow | Xcode → Scheme → Diagnostics     | ★★★★★         |
| **Thread Sanitizer**              | Data Race (косвенно помогает)          | Xcode → Diagnostics              | ★★★★☆         |
| **Zombies** (NSZombies)           | Use After Free в Objective-C объектах  | Xcode → Diagnostics → Enable Zombie Objects | ★★★★☆         |
| **Instruments → Zombies**         | UAF в runtime                          | Instruments → Zombies            | ★★★★★         |
| **Swift 6 Strict Concurrency**    | Dangling pointer / unsafe передача     | Build Settings → Swift Compiler  | ★★★★★         |
| **Xcode Memory Graph Debugger**   | Подозрительные retain cycles + UAF    | Debug navigator → Memory Graph   | ★★★★☆         |

### 6. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит dangling pointer и unsafe передачу  
- **weak вместо unowned** — используйте `weak` везде, где объект может исчезнуть раньше  
- **defer** — спасает при работе с UnsafePointer, CFRelease, malloc/free  
- **actor** — для любого изменяемого состояния (нет прямого доступа → нет UAF)  
- **Sendable** — всё, что передаётся между акторами, должно быть безопасным  
- **Unsafe API** — используйте только когда **нет альтернативы** (Metal, Accelerate, C API)  
- **Unmanaged** — контролируйте retain/release вручную, но с осторожностью  
- **Для тестов** — включайте Address Sanitizer + Zombies + Thread Sanitizer  
- **Для legacy-кода** — постепенно заменяйте unowned на weak, Unsafe на безопасные альтернативы

**Короткий девиз 2026**:
> «Use After Free — это когда ты обращаешься к могиле после похорон.  
> В 2026 году ответ один: weak вместо unowned + actor + strict concurrency + Address Sanitizer.  
> Unsafe, unowned, ручное управление памятью — это уже legacy. Забудьте про них в новом коде.»
