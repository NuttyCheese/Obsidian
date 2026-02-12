#objective_c #objc_runtime #ios #dynamic_runtime #selectors #method_swizzling #reflection #message_sending #class_inspection #runtime_features
**Objective-C Runtime** — это фундаментальная часть [[Objective-C]], которая обеспечивает **динамическую типизацию**, **динамическую загрузку классов**, **рефлексию** и **диспетчеризацию методов** во время выполнения программы.  
Она существует как отдельная библиотека (`libobjc`) и до сих пор активно используется в [[iOS]]/macOS, даже в чистом [[Swift]]-коде (через мост к [[Foundation]]/[[UIKit]]/AppKit).

### 1. Основные возможности Objective-C Runtime

| Возможность                     | Описание                                                             | Пример использования в 2026 году         |
| ------------------------------- | -------------------------------------------------------------------- | ---------------------------------------- |
| Динамическая загрузка классов   | Классы могут регистрироваться в runtime во время выполнения          | Swizzling, плагины, модульность          |
| Динамическое добавление методов | `class_addMethod`, `class_replaceMethod`                             | Method [[swizzling]] (AOP, тестирование) |
| Интроспекция (рефлексия)        | `class_getInstanceMethod`, `class_copyMethodList`, `object_getClass` | Отладка, инспекторы, сериализация        |
| Ассоциативные объекты           | `objc_setAssociatedObject`, `objc_getAssociatedObject`               | Хранение данных без субклассинга         |
| Динамическая диспетчеризация    | Вызов через `objc_msgSend`                                           | Основной механизм вызова методов         |
| Forwarding сообщений            | `forwardInvocation:`, `resolveInstanceMethod:`                       | Message forwarding, прокси-объекты       |

### 2. Диспетчеризация методов ([[Method Dispatch]]) в Objective-C Runtime

**objc_msgSend** — сердце динамической диспетчеризации.

Процесс (упрощённо):

1. Объект получает сообщение (селектор, например `@selector(doSomething)`)
2. Runtime ищет реализацию метода в **dispatch table** класса объекта
3. Если метод найден → вызов
4. Если нет → идёт вверх по иерархии наследования
5. Если не найден нигде → вызывается `resolveInstanceMethod:`, затем `forwardingTargetForSelector:`, затем `forwardInvocation:`, затем `doesNotRecognizeSelector:`

### 3. Dispatch Table (таблица методов)

Каждый класс имеет **таблицу методов** (method list + cache):

- **Методы** хранятся в `class_rw_t` → `method_list_t`
- **Кэш** (`method_cache_t`) — ускоряет повторные вызовы (hash table селектор → IMP)
- **IMP** — указатель на реализацию метода (function pointer)

```objc
// Пример структуры (упрощённо)
struct method_t {
    SEL name;           // селектор (@selector)
    const char *types;  // типы аргументов и возвращаемого значения
    IMP imp;            // указатель на функцию
};
```

### 4. Сравнение диспетчеризации в Swift и Objective-C Runtime

| Аспект                        | Objective-C Runtime (objc_msgSend)          | Swift (2026)                                |
|-------------------------------|---------------------------------------------|---------------------------------------------|
| Тип диспетчеризации           | Только динамическая                         | Статическая / Динамическая / Протокольная |
| Скорость                      | Медленная (поиск + кэш)                     | Очень быстрая (статическая) / средняя (динамическая) |
| Полиморфизм                   | Полный, всегда динамический                 | Только при вызове через `class` / `any`     |
| Swizzling                     | Легко (`method_exchangeImplementations`)    | Сложнее, требует `@objc` и `objc-runtime`  |
| Forwarding                    | `forwardInvocation:`                        | `forwardingTarget(for:)` / `doesNotRecognizeSelector:` |
| Использование в чистом Swift  | Да (через `@objc`, Foundation)              | Редко, только для ObjC-интеропа             |

### 5. Реальные примеры использования [[Runtime]] в 2026 году

#### Пример 1 — Method Swizzling (AOP)

```swift
import ObjectiveC.runtime

extension UIViewController {
    static func swizzleViewDidLoad() {
        let original = #selector(UIViewController.viewDidLoad)
        let swizzled = #selector(UIViewController.swizzled_viewDidLoad)
        
        let originalMethod = class_getInstanceMethod(UIViewController.self, original)!
        let swizzledMethod = class_getInstanceMethod(UIViewController.self, swizzled)!
        
        method_exchangeImplementations(originalMethod, swizzledMethod)
    }
    
    @objc func swizzled_viewDidLoad() {
        print("ViewController \(type(of: self)) загружен")
        swizzled_viewDidLoad()  // вызов оригинала
    }
}
```

#### Пример 2 — Динамическое добавление метода

```swift
func dynamicMethod() {
    print("Динамически добавленный метод!")
}

class DynamicClass: NSObject {}

let sel = #selector(dynamicMethod)
class_addMethod(DynamicClass.self, sel, imp_implementationWithBlock(dynamicMethod as @convention(block) () -> Void), "v@:")
```

#### Пример 3 — Ассоциативные объекты (без субклассинга)

```swift
private var AssociatedKey: UInt8 = 0

extension UIView {
    var customTag: String? {
        get { objc_getAssociatedObject(self, &AssociatedKey) as? String }
        set { objc_setAssociatedObject(self, &AssociatedKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC) }
    }
}
```

### 6. Короткий итог (2026)

- **Objective-C Runtime** — это **низкоуровневая машина** динамики ObjC
- Основной движок — `objc_msgSend` + dispatch table
- В чистом Swift используется **редко** — только для ObjC-интеропа, swizzling, рефлексии
- Самые частые применения сегодня:
  - Method swizzling (тестирование, логирование, AOP)
  - Ассоциативные объекты
  - Динамическое добавление методов
  - Интроспекция (runtime-инспекторы, отладка)

**Главное правило**:
> «В 2026 году используй Runtime только когда **нельзя** решить задачу чистым Swift.  
> Чем меньше Runtime-кода — тем лучше поддержка, производительность и читаемость.»
