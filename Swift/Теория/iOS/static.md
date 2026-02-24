**`static`** — это ключевое слово в [[Swift]], которое используется в нескольких разных контекстах. Ниже максимально подробный разбор всех основных значений и сценариев использования `static` в 2026 году (Swift 5.10+ и Swift 6).

### 1. static в структурах, классах и перечислениях (самый частый случай)

`static` объявляет **статический** член типа — свойство, метод, сабскрипт или вложенный тип, который **принадлежит самому типу**, а не конкретному экземпляру.

```swift
struct MathConstants {
    static let pi = 3.141592653589793
    static let e  = 2.718281828459045
    
    static func square(_ number: Double) -> Double {
        number * number
    }
    
    static var goldenRatio: Double {
        (1 + sqrt(5)) / 2
    }
}

// Использование — через имя типа
print(MathConstants.pi)           // 3.141592653589793
print(MathConstants.square(5))    // 25.0
print(MathConstants.goldenRatio)  // ≈ 1.618033988749895
```

**Ключевые правила**:
- `static` свойства и методы **не требуют создания экземпляра**
- `static` свойства **не могут** быть [[lazy]] (в отличие от instance-вычисляемых)
- `static` [[let]] — константа (неизменяемая)
- `static` [[var]] — может быть изменяемой

### 2. static vs class (разница между static и class)

| Характеристика                        | static                                       | class                                         | Когда выбирать в 2026                                |
| ------------------------------------- | -------------------------------------------- | --------------------------------------------- | ---------------------------------------------------- |
| Может быть переопределён в подклассе? | Нет                                          | Да                                            | [[class]] — если нужна полиморфная переопределимость |
| Применяется к                         | [[struct]], [[enum]], class                  | только class                                  | —                                                    |
| Пример переопределения                | Невозможно                                   | Можно                                         | class — для иерархий классов                         |
| Хранение                              | В метаданных типа                            | В метаданных типа                             | —                                                    |
| Использование в протоколах            | Можно (static в протоколе = требование типа) | Можно (class в протоколе = требование класса) | static — чаще                                        |

Пример разницы:

```swift
class Animal {
    class func sound() -> String { "..." }      // можно переопределить
    static func kingdom() -> String { "Animalia" } // нельзя переопределить
}

class Dog: Animal {
    override class func sound() -> String { "Woof!" }
    // override static func kingdom() -> String { ... } // ошибка компиляции
}
```

### 3. static в протоколах (требование реализации на уровне типа)

```swift
protocol Identifiable {
    static var identifier: String { get }
}

struct User: Identifiable {
    static let identifier = "user"
}

enum Role: Identifiable {
    static let identifier = "role"
}
```

Это **очень популярный** паттерн для:
- Reuse Identifier в [[UITableView]]/[[UICollectionView]]
- Cell registration
- Dependency Injection (типы как ключи)

### 4. static в расширениях (самый частый способ добавления "утилит")

```swift
extension UIColor {
    static let appPrimary   = UIColor(red: 0.2, green: 0.4, blue: 0.8, alpha: 1)
    static let appAccent    = UIColor.systemPurple
    static let appError     = UIColor.systemRed
    
    static func randomPastel() -> UIColor {
        UIColor(
            hue: CGFloat.random(in: 0...1),
            saturation: 0.4,
            brightness: 0.95,
            alpha: 1
        )
    }
}

// Использование
view.backgroundColor = .appPrimary
let pastel = UIColor.randomPastel()
```

### 5. static let vs static var vs static computed var

```swift
struct Constants {
    static let appVersion     = "2.5.1"                     // константа
    static var buildNumber    = 142                         // изменяемая переменная
    static var currentUserId: String? {                     // вычисляемое
        UserDefaults.standard.string(forKey: "currentUserId")
    }
}
```

### 6. static в [[enum]] (часто для связанных значений)

```swift
enum NetworkEnvironment {
    case development, staging, production
    
    static let current: NetworkEnvironment = .development
    
    static var baseURL: URL {
        switch current {
        case .development: return URL(string: "https://dev.api.com")!
        case .staging:     return URL(string: "https://staging.api.com")!
        case .production:  return URL(string: "https://api.com")!
        }
    }
}
```

### 7. static в глобальном контексте (глобальные утилиты)

```swift
enum Logger {
    static func info(_ message: String) {
        print("[INFO] \(message)")
    }
    
    static func error(_ message: String, error: Error?) {
        print("[ERROR] \(message)", error?.localizedDescription ?? "")
    }
}

// Использование
Logger.info("Пользователь вошёл")
```

### Лучшие практики static в Swift 2026

- **Предпочитайте** `static let` для констант — они thread-safe и оптимизированы  
- **Используйте** `static var` только если значение должно меняться (редко)  
- **Для утилит** — создавайте `enum` с `static` методами и свойствами (без `case`) — это самый чистый паттерн  
- **Для Reuse Identifier** — всегда `static let` в ячейке/вью  
- **Для цветов/шрифтов** — `static` в `extension UIColor` / `extension UIFont`  
- **Избегайте** `static var` с мутабельным состоянием в больших масштабах — это источник багов и сложностей с тестированием  
- **В SwiftUI** — `static` часто используется в `View` для констант и утилит  
- **Документируйте** — пишите комментарий «static let reuseIdentifier — идентификатор ячейки для регистрации в таблице»

**Короткий итог 2026**:
> `static` — это модификатор, который делает член **принадлежащим типу**, а не экземпляру.  
> В 2026 году:  
> - самый частый кейс — `static let` константы и `static func` утилиты  
> - `static` vs `class` — `class` только для переопределяемых методов в классах  
> - в протоколах — `static` требует реализации на уровне типа  
> - в расширениях — идеально для цветов, шрифтов, утилит  
> Это **один из самых мощных** и **самых часто используемых** инструментов в Swift.
