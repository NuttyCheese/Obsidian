#crash #warning #xcode #swift
### Что это значит

[[Xcode]] предупреждает, что вы пытаетесь использовать **[[API]], недоступный в [[Swift]]**.

- Такие API могут существовать в [[Objective-C]], но **не были импортированы в Swift** или помечены как `NS_SWIFT_UNAVAILABLE`.
    
- Использование этих методов приведёт к ошибкам компиляции.
    
- Обычно это связано с устаревшими или специфичными для Objective-C API, которые **не имеют прямого аналога в Swift**.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Использование Objective-C метода, недоступного в Swift**

```objc
// Objective-C
@interface MyClass : NSObject
- (void)doSomethingOld NS_SWIFT_UNAVAILABLE("Use doSomethingNew instead");
@end
```

```swift
let obj = MyClass()
obj.doSomethingOld() // ⚠️ 'doSomethingOld()' is unavailable in Swift
```

- Метод помечен как недоступный в Swift → компилятор выдаёт ошибку.
    

---

**Пример 2: Использование устаревшего API**

```objc
@interface UIView (Deprecated)
@property (nonatomic, strong) IBOutlet UIView *oldOutlet NS_SWIFT_UNAVAILABLE("Use newOutlet instead");
@end
```

```swift
let view = UIView()
view.oldOutlet // ⚠️ 'oldOutlet' is unavailable in Swift
```

- Недоступен для Swift → нужно использовать новый API.
    

---

### Как исправить

#### 1️⃣ Использовать альтернативный Swift API

```swift
let obj = MyClass()
obj.doSomethingNew()  // ✅ метод доступен в Swift
```

- Чаще всего Apple предоставляет **новый метод или свойство**, заменяющее недоступный.
    

---

#### 2️⃣ Если нужно использовать Objective-C API напрямую

- Можно создать **обёртку ([[wrapper]])** на Objective-C, доступную в Swift:
    

```objc
// MyWrapper.h
@interface MyWrapper : NSObject
- (void)doSomethingSafe;
@end

// MyWrapper.m
@implementation MyWrapper
- (void)doSomethingSafe {
    [[MyClass new] doSomethingOld]; // используем Objective-C внутри обёртки
}
@end
```

```swift
let wrapper = MyWrapper()
wrapper.doSomethingSafe() // ✅ безопасно вызываем из Swift
```

---

#### 3️⃣ Проверять документацию Apple

- Недоступные методы обычно имеют рекомендацию **альтернативного API для Swift**.
    
- Использование `#available` не помогает, если метод полностью `unavailable`.
    

---

### Резюме

- Предупреждение возникает при попытке использовать **Objective-C API, недоступный в Swift**.
    
- Исправляется использованием **современного Swift API**, созданием обёртки или изучением рекомендаций Apple.
    
- Обеспечивает корректность компиляции и предотвращает runtime проблемы.
    

---
