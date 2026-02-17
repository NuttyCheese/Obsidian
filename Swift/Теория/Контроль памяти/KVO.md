**KVO** — механизм наблюдения за изменениями свойств объектов в [[Runtime]].  
Позволяет автоматически реагировать на изменение `@objc dynamic` свойств.

### Ключевые особенности (2026)

- Работает **только** с классами, наследующими [[NSObject]]
- Требует `@objc dynamic` для наблюдаемых свойств
- Основан на **Objective-C Runtime**
- Поддерживает **несколько наблюдателей** на одно свойство
- **Не** работает с [[struct]], [[enum]], чистым Swift-типами

### Основные методы и варианты использования

#### 1. Классический KVO через `observe(_:options:changeHandler:)`

```swift
class Person: NSObject {
    @objc dynamic var name: String = "Anonymous"
    @objc dynamic var age: Int = 0
}

let person = Person()

// Наблюдение за name
let nameObservation = person.observe(\.name, options: [.old, .new]) { person, change in
    print("Name changed: \(change.oldValue ?? "nil") → \(change.newValue ?? "nil")")
}

// Наблюдение за age (только новое значение + начальное)
let ageObservation = person.observe(\.age, options: [.initial, .new]) { _, change in
    print("Age is now: \(change.newValue ?? 0)")
}

person.name = "Alice"    // → Name changed: Anonymous → Alice
person.age = 28          // → Age is now: 0  (initial)
                         // → Age is now: 28
```

#### 2. Хранение наблюдателей (рекомендуемый паттерн)

```swift
class ProfileViewController: UIViewController {
    private var observations: [NSKeyValueObservation] = []
    
    private let user = User()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        observations.append(
            user.observe(\.name, options: [.new]) { [weak self] _, change in
                self?.updateNameLabel(change.newValue ?? "")
            }
        )
        
        observations.append(
            user.observe(\.isPremium, options: [.new]) { [weak self] _, change in
                self?.updatePremiumBadge(change.newValue ?? false)
            }
        )
    }
    
    deinit {
        observations.forEach { $0.invalidate() }
    }
}
```

#### 3. KVO для [[UIKit]]-свойств (часто используется)

```swift
class CustomView: UIView {
    private var observations: [NSKeyValueObservation] = []
    
    override func didMoveToSuperview() {
        super.didMoveToSuperview()
        
        observations.append(
            superview?.observe(\.backgroundColor, options: [.new]) { [weak self] _, change in
                guard let color = change.newValue as? UIColor else { return }
                self?.layer.borderColor = color.cgColor
            }
        )
    }
    
    deinit {
        observations.forEach { $0.invalidate() }
    }
}
```

#### 4. KVO + [[Combine]] (мост между старым и новым)

```swift
import Combine

class ViewModel {
    @Published var counter = 0
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // Старый KVO → Combine
        let obj = LegacyObject()
        obj.publisher(for: \.value, options: [.new])
            .sink { [weak self] newValue in
                self?.counter = newValue
            }
            .store(in: &cancellables)
    }
}
```

#### 5. KVO для нескольких свойств сразу (KeyPath)

```swift
class Settings: NSObject {
    @objc dynamic var theme: String = "light"
    @objc dynamic var fontSize: CGFloat = 17.0
}

let settings = Settings()

let observation = settings.observe(\.theme, \.fontSize) { settings, _ in
    print("Theme: \(settings.theme), Font: \(settings.fontSize)")
}

settings.theme = "dark"     // сработает
settings.fontSize = 19      // сработает
```

#### 6. KVO + Computed свойства (через @objc dynamic)

```swift
class Calculator: NSObject {
    @objc dynamic var a: Double = 0
    @objc dynamic var b: Double = 0
    
    @objc dynamic var sum: Double {
        a + b
    }
}

let calc = Calculator()

calc.observe(\.sum) { calc, _ in
    print("Sum changed to: \(calc.sum)")
}

calc.a = 5      // → Sum changed to: 5
calc.b = 7      // → Sum changed to: 12
```

### Когда использовать KVO в 2026 году

| Ситуация                                      | Рекомендация                       | Альтернатива (современная) |
| --------------------------------------------- | ---------------------------------- | -------------------------- |
| Работа с legacy UIKit-кодом                   | KVO всё ещё актуален               | Combine / @Published       |
| Наблюдение за свойствами [[NSObject]]-классов | KVO — единственный нативный способ | —                          |
| Динамическое связывание UIKit → ViewModel     | KVO + Combine                      | SwiftUI + @ObservedObject  |
| Поддержка старых проектов                     | KVO                                | Миграция на Combine        |
| Новый проект на UIKit                         | Combine / @Published               | SwiftUI                    |

### Плюсы и минусы KVO (2026)

**Плюсы:**
- Работает с любым NSObject и `@objc dynamic`
- Поддерживает старый UIKit-код
- Мощная опция `.initial`, `.prior`, `.old`, `.new`

**Минусы:**
- Требует `NSObject` + `@objc dynamic`
- Легко забыть `invalidate()` → утечка памяти
- Медленнее Combine / SwiftUI
- Нет встроенной поддержки async / await

### Итог — современное правило

> «В 2026 году используй KVO **только** если работаешь с legacy UIKit или NSObject-подклассами без возможности миграции.  
> Для всего нового — **Combine**, **@Published**, **SwiftUI @State / @ObservedObject / @EnvironmentObject**.»
