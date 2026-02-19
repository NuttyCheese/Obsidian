**`NSObject`** — это **базовый класс** всей иерархии классов в Objective-C и UIKit/Foundation в iOS/macOS.

В Swift он остаётся **самым важным родительским классом** для всех объектов, написанных на Objective-C runtime (включая почти весь UIKit, Foundation, AppKit и т.д.).

### 1. Почему NSObject всё ещё важен в 2025–2026

Даже если вы пишете чистый Swift-код, `NSObject` появляется в следующих ситуациях:

| Сценарий                                      | Почему нужен NSObject                                  | Пример в 2026 |
|-----------------------------------------------|--------------------------------------------------------|---------------|
| Любой класс, который наследуется от UIKit     | `UIView`, `UIViewController`, `UILabel`, `UIButton` и т.д. | 99% UIKit-приложений |
| Использование KVO (Key-Value Observing)       | KVO работает только с классами, наследующими `NSObject` | `@objc dynamic var` |
| Делегирование (delegates) в UIKit             | `UITableViewDelegate`, `UITextFieldDelegate` и т.д.    | Почти все UIKit-контроллеры |
| `performSelector`, `responds(to:)`            | Динамическая диспетчеризация Objective-C               | Редко, но в legacy-коде |
| `NSKeyedArchiver` / `NSCoding`                | Сериализация старым способом                           | Старые данные |
| `NotificationCenter` с `object:`              | `object` обычно `NSObject`                             | Системные уведомления |
| `Selector` и `#selector`                      | Работают только с методами `NSObject`-классов          | `@objc func` |
| `associated objects` (objc_setAssociatedObject) | Хранение данных в runtime-ассоциациях                 | Редко, но мощно |

### 2. Ключевые возможности NSObject (что реально используется)

| Метод / Свойство / Возможность              | Что делает                                              | Пример использования 2026 |
|---------------------------------------------|---------------------------------------------------------|---------------------------|
| `class func` / `static func`                | Методы класса                                           | `UIView.appearance()`     |
| `@objc` / `dynamic`                         | Делает метод/свойство видимым для Obj-C runtime        | KVO, delegates            |
| `responds(to:)`                             | Проверяет, реализует ли объект селектор                 | Редко, но в legacy        |
| `perform(_:with:)`                          | Динамический вызов метода                               | Очень редко (опасно)      |
| `value(forKey:)` / `setValue(_:forKey:)`    | KVC (Key-Value Coding)                                  | Редко, в legacy           |
| `observeValue(forKeyPath:of:change:context:)` | KVO-обработка                                        | `@objc dynamic` свойства  |
| `willChangeValue(forKey:)` / `didChangeValue(forKey:)` | Ручное KVO                                          | Редко                     |
| `objc_setAssociatedObject` / `objc_getAssociatedObject` | Хранение данных в runtime-ассоциациях               | Расширение UIKit-классов  |

### 3. Самый популярный паттерн 2026: NSObject + @objc + KVO

```swift
class ViewModel: NSObject {
    @objc dynamic var isLoading = false {
        didSet {
            // KVO будет уведомлять всех наблюдателей
        }
    }
    
    func fetchData() {
        isLoading = true
        // ...
        isLoading = false
    }
}

// Подписка на KVO
viewModel.addObserver(self, forKeyPath: \.isLoading, options: [.new], context: nil)

override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
    if keyPath == \ViewModel.isLoading {
        if let isLoading = change?[.newKey] as? Bool {
            updateUI(isLoading: isLoading)
        }
    }
}
```

### 4. Лучшие практики NSObject в Swift 2026

- **Не наследуйтесь** от `NSObject` без необходимости  
  → Если пишешь чистый Swift — используй `struct`, `class` без `NSObject`, `final class`  
- **Наследуйтесь** от `NSObject` только когда:
  - нужен KVO (`@objc dynamic`)
  - класс реализует делегат UIKit/AppKit
  - нужен `associated objects`
  - класс участвует в Objective-C runtime (selectors, performSelector)

- **Используй `final class`** → если наследование не нужно, компилятор оптимизирует вызовы  
- **Используй `AnyObject`** вместо `NSObject` для ограничений протоколов  
- **В SwiftUI** — почти никогда не нужен `NSObject` (кроме редких UIKit-обёрток)  
- **Swift 6 strict concurrency** — классы, наследующие `NSObject`, **не** `Sendable` по умолчанию — помечай `@MainActor` или делай `final class` + `Sendable` вручную  
- **Документируйте** — пиши комментарий «NSObject — нужен для KVO и delegates UIKit»

**Короткий девиз 2026**:
> `NSObject` — это **мост** между старым Objective-C миром (UIKit, Foundation) и современным Swift.  
> В 2026 году:  
> - наследуй от `NSObject` **только** если нужен KVO, delegates, associated objects или Obj-C runtime  
> - в чистом Swift → избегай `NSObject` (используй `struct`, `final class`, `AnyObject`)  
> - `final class` + `@objc dynamic` — стандарт для KVO  
> - в SwiftUI — почти никогда не нужен  
> Это **наследие**, а не основа нового кода.

Удачи с чистым и современным кодом без лишнего наследования от NSObject! 🚀