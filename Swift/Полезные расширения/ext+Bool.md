#extension #bool #int #array 

---
# [[Swift]] — Полезные расширения для [[Bool]], [[Int]] и [[Array]]<Bool>

Небольшие, но очень часто используемые расширения, которые упрощают работу с булевыми значениями, их преобразованиями и локализацией.

## 1. Bool → числовые типы и [[NSNumber]]

```swift
extension Bool {
    /// Bool → Int (1 / 0) — самый частый сценарий
    var toInt: Int { self ? 1 : 0 }
    
    var int8Value: Int8   { self ? 1 : 0 }
    var int16Value: Int16 { self ? 1 : 0 }
    var int32Value: Int32 { self ? 1 : 0 }
    var int64Value: Int64 { self ? 1 : 0 }
    
    var floatValue: Float  { self ? 1.0 : 0.0 }
    var doubleValue: Double { self ? 1.0 : 0.0 }
    
    /// Для совместимости с Objective-C / старыми API
    var numberValue: NSNumber { NSNumber(value: self.toInt) }
}
```

**Зачем это нужно**

- Передача в [[API]], которые ожидают `Int` вместо `Bool` (особенно старые backend'и)
- Запись в [[UserDefaults]] / [[JSON]] где `Bool` не поддерживается
- Работа с [[Core Data]] / Realm (некоторые поля хранятся как Int)
- Вычисление сумм / процентов от массива булевых значений

**Примеры**

```swift
let isActive = true
let param: [String: Any] = [
    "active": isActive.toInt,           // 1
    "score": isActive.doubleValue * 100 // 100.0
]

UserDefaults.standard.set(isActive.numberValue, forKey: "wasLaunched")
```

## 2. Int → Bool

```swift
extension Int {
    /// Int → Bool (всё кроме 0 = true)
    var boolValue: Bool { self != 0 }
}
```

**Типичные случаи**

- Парсинг ответа от сервера, где флаг приходит как `1`/`0` / любое ненулевое значение
- Чтение из старых баз данных / plist'ов

**Пример**

```swift
let serverFlag = json["premium"] as? Int ?? 0
if serverFlag.boolValue {
    unlockPremiumFeatures()
}
```

## 3. Bool → локализованные строки

```swift
extension Bool {
    var localizedYesNo: String {
        self ? NSLocalizedString("yes", comment: "")
             : NSLocalizedString("no", comment: "")
    }
    
    var localizedEnabled: String {
        self ? NSLocalizedString("enabled", comment: "")
             : NSLocalizedString("disabled", comment: "")
    }
}
```

**Зачем**

- Красивый вывод в UI без тернарных операторов
- Легко поддерживать несколько языков (если в .strings есть "yes", "no", "enabled", "disabled")

**Пример**

```swift
statusLabel.text = isOnline.localizedYesNo.uppercased()
// → "YES" или "NO"

switchLabel.text = "Wi-Fi: \(isWiFiOn.localizedEnabled)"
// → "Wi-Fi: Enabled" / "Wi-Fi: Disabled"
```

**Рекомендация**: лучше вынести ключи в enum, чтобы не плодить «магические строки»

```swift
enum LocalizedBool: String {
    case yes     = "yes"
    case no      = "no"
    case enabled = "enabled"
    case disabled = "disabled"
}

extension Bool {
    var localized: String {
        NSLocalizedString(self ? LocalizedBool.yes.rawValue : LocalizedBool.no.rawValue, comment: "")
    }
}
```

## 4. Array<Bool> — полезная статистика и преобразования

```swift
extension Array where Element == Bool {
    /// Количество true
    var trueCount: Int { self.filter { $0 }.count }
    
    /// Количество false
    var falseCount: Int { self.filter { !$0 }.count }
    
    /// Все элементы true?
    var allTrue: Bool { !self.contains(false) }
    
    /// Все элементы false?
    var allFalse: Bool { !self.contains(true) }
    
    /// Преобразование в [Int] (1 / 0)
    func toIntArray() -> [Int] {
        self.map { $0.toInt }
    }
}
```

**Типичные сценарии**

- Подсчёт выполненных заданий / отмеченных чекбоксов
- Проверка, все ли условия выполнены (в форме, валидация)
- Передача массива флагов в аналитику / сервер

**Примеры**

```swift
let answers = [true, false, true, true, false]
print("Правильных: \(answers.trueCount)")     // 3
print("Все верно? \(answers.allTrue)")        // false

let allAgreed = termsCheckboxes.allTrue
if allAgreed {
    enableSignUpButton()
}

let analyticsFlags = checkboxes.toIntArray()  // [1, 0, 1, 0]
trackEvent("checkboxes", parameters: ["values": analyticsFlags])
```

## Полный обзор — когда что использовать

| Свойство / Метод         | Возвращает     | Самый частый сценарий                          |
|---------------------------|----------------|------------------------------------------------|
| `toInt`                   | `Int`          | JSON, API, UserDefaults, аналитика             |
| `numberValue`             | `NSNumber`     | [[Objective-C]] bridging, старые фреймворки        |
| `boolValue` (на Int)      | `Bool`         | Парсинг ответа сервера (1/0 → true/false)      |
| `localizedYesNo`          | `String`       | Текстовый вывод в UILabel / Text               |
| `allTrue` / `allFalse`    | `Bool`         | Валидация форм, чек-листов                     |
| `trueCount`               | `Int`          | Статистика, прогресс-бары                      |
| `toIntArray()`            | `[Int]`        | Передача массива флагов дальше                 |

## Рекомендуемые дополнения (что часто добавляют)

```swift
extension Bool {
    /// Инверсия с chaining
    var toggled: Bool { !self }
    
    /// Для SwiftUI / Combine
    var asPublisher: CurrentValueSubject<Bool, Never> {
        .init(self)
    }
}

extension Collection where Element == Bool {
    /// Процент выполненных (0...1)
    var completionRatio: Double {
        guard !isEmpty else { return 0 }
        return Double(trueCount) / Double(count)
    }
}
```
