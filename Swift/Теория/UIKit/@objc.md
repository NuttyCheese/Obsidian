**@objc** — это атрибут в Swift, который делает объявленный элемент (класс, метод, свойство, протокол) **видимым и доступным** для **Objective-C [[runtime]]**.

Без `@objc` Swift-элемент остаётся «закрытым» внутри [[Swift]] и недоступен для Objective-C кода, селекторов, [[KVO]], Interface Builder и старого [[Objective-C]] [[API]].

### Зачем нужен @objc в 2026 году (реальные сценарии)

| Сценарий                                      | Почему нужен именно @objc                              | Пример |
|-----------------------------------------------|--------------------------------------------------------|--------|
| Использование `#selector`                     | Селекторы — это Objective-C механизм                   | `button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)` |
| `@IBAction` и `@IBOutlet` в Interface Builder | IB работает через Objective-C runtime                  | `@IBAction func tapped(_ sender: UIButton)` |
| Key-Value Observing (KVO)                     | KVO — чисто Objective-C фича                           | `@objc dynamic var age: Int` |
| Опциональные методы в протоколах              | Objective-C протоколы поддерживают optional методы     | `@objc optional func optionalMethod()` |
| Вызов из Objective-C кода                     | Старый код / библиотеки на ObjC                        | Редко, но встречается в legacy |
| `Timer`, `NotificationCenter`, `performSelector` | Все эти API используют селекторы                      | `Timer.scheduledTimer(timeInterval: 1, target: self, selector: #selector(tick), ...)` |
| Динамический вызов методов                    | `performSelector`, `responds(to:)` и т.д.              | Редко, но нужно для низкоуровневой магии |

### Ключевые правила и синтаксис @objc

```swift
// 1. Метод с селектором
@objc func buttonTapped(_ sender: UIButton) { ... }

// 2. Свойство для KVO
@objc dynamic var counter: Int = 0

// 3. Класс целиком (все публичные методы/свойства становятся @objc)
@objc class MyClass: NSObject { ... }

// 4. Протокол с optional методами
@objc protocol MyDelegate {
    @objc optional func optionalMethod()
}

// 5. Переименовать селектор (редко, но полезно)
@objc(buttonWasTapped:)
func tappedButton(_ sender: UIButton) { ... }
```

### Самые важные нюансы 2026 года

- **@objc требует наследования от NSObject**  
  Без `NSObject` (или класса, наследующего [[NSObject]]) `@objc` не скомпилируется.

- **@objc + dynamic** — для KVO и полного динамического dispatch  
  `@objc` делает метод видимым, `dynamic` — заставляет использовать динамический dispatch (через runtime) вместо статического.

```swift
@objc dynamic var name: String = ""  // KVO будет работать
```

- **private @objc** — разрешено, но селектор всё равно будет доступен  
  Можно использовать `#selector(MyClass.privateMethod)` даже если метод private.

- **@objc без параметров** — для свойств и методов без аргументов  
  `@objc var isEnabled: Bool`

- **Swift 6 strict concurrency**  
  `@objc` методы вызываются на главном потоке → безопасны, но если внутри async — используй `Task { @MainActor in ... }`

### Короткий чек-лист «Когда писать @objc»

- Нужно `#selector` — да  
- `@IBAction` / `@IBOutlet` — да (автоматически добавляется)  
- KVO (`observe(\.keyPath)`) — да + `dynamic`  
- Optional протоколы Objective-C — да  
- `Timer`, `performSelector`, `NotificationCenter` с селектором — да  
- Вызов из Objective-C кода — да  
- Чистый Swift без ObjC-runtime — **нет**

**Короткий девиз 2026**:
> @objc — это «мост» из Swift в Objective-C runtime.  
> Пиши его, когда нужен селектор, KVO, Interface Builder, [[optional]] протоколы Objective-C или старый ObjC-код.  
> В 2026 году @objc нужен везде, где есть #selector, [[@IBAction]], KVO или Objective-C API.  
> Без него Swift-код остаётся «невидимым» для Objective-C мира.
