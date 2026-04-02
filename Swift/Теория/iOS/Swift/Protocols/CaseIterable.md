#swift #enum #caseiterable #protocol #iteration #collections

---
### Определение
**`CaseIterable`** — это протокол в Swift, который предоставляет коллекцию всех возможных кейсов перечисления ([[enum]]). Типы, соответствующие `CaseIterable`, получают автоматически синтезированное свойство `allCases` — массив, содержащий все кейсы перечисления в том порядке, в котором они объявлены.

Этот протокол особенно полезен для перечислений, представляющих конечный набор дискретных значений (например, дни недели, варианты выбора, типы действий), когда нужно перебрать все возможные варианты.

### Зачем это знать iOS-разработчику?
1.  **Итерация по всем кейсам:** Легко получить массив всех значений перечисления для отображения в `Picker`, `List` или `SegmentedControl`.
2.  **Генерация данных:** Создание тестовых данных или заполнение выпадающих списков.
3.  **Валидация и проверки:** Проверка, что значение находится в допустимом наборе.
4.  **Синтез компилятором:** Для простых перечислений без ассоциированных значений компилятор автоматически генерирует `allCases`.
5.  **Динамическая типизация:** Использование в дженериках для работы с любым `CaseIterable` типом.

---

### Базовый синтаксис

#### Простое перечисление (синтезированная реализация)

```swift
enum Direction: CaseIterable {
    case north, south, east, west
}

let allDirections = Direction.allCases
print(allDirections)  // [north, south, east, west]
print(allDirections.count)  // 4

for direction in Direction.allCases {
    print(direction)
}
// north
// south
// east
// west
```

#### Перечисление с raw значениями

```swift
enum Planet: Int, CaseIterable {
    case mercury = 1, venus, earth, mars
}

print(Planet.allCases.count)  // 4
print(Planet.allCases.map { $0.rawValue })  // [1, 2, 3, 4]
```

---

### Когда компилятор синтезирует `allCases` автоматически

Компилятор автоматически синтезирует `allCases` для перечислений, которые:

1.  **Не имеют ассоциированных значений** ([[associated value]]s).
2.  **Не являются рекурсивными** (indirect).
3.  **Не используют `@available`** на отдельных кейсах (все кейсы должны быть доступны).

```swift
// ✅ Синтезируется
enum Simple: CaseIterable {
    case a, b, c
}

// ❌ Не синтезируется (есть ассоциированные значения)
enum WithAssociated: CaseIterable {
    case a(Int)
    case b(String)
    // Error: Type 'WithAssociated' does not conform to protocol 'CaseIterable'
}

// ❌ Не синтезируется (indirect)
indirect enum Tree: CaseIterable {
    case leaf
    case node(Tree, Tree)
}

// ❌ Не синтезируется (разные доступности)
enum Versioned: CaseIterable {
    case v1
    @available(*, deprecated)
    case v2
    // Error: All cases of 'Versioned' must be available to use 'CaseIterable'
}
```

---

### Ручная реализация `CaseIterable`

Если компилятор не может синтезировать `allCases`, вы можете реализовать его вручную.

#### Перечисление с ассоциированными значениями

```swift
enum NetworkStatus: CaseIterable {
    case connected(speed: Double)
    case disconnected
    case connecting
    
    static var allCases: [NetworkStatus] {
        return [.connecting, .disconnected, .connected(speed: 100.0)]
    }
}

for status in NetworkStatus.allCases {
    print(status)
}
// connecting
// disconnected
// connected(speed: 100.0)
```

#### Перечисление с `indirect`

```swift
indirect enum BinaryTree: CaseIterable {
    case leaf
    case node(BinaryTree, BinaryTree)
    
    static var allCases: [BinaryTree] {
        return [.leaf, .node(.leaf, .leaf)]
    }
}

print(BinaryTree.allCases.count)  // 2
```

#### Перечисление с динамическими кейсами

```swift
enum ColorPalette: CaseIterable {
    case red, green, blue
    
    static var allCases: [ColorPalette] {
        // Динамически определяем кейсы (например, из UserDefaults)
        let stored = UserDefaults.standard.array(forKey: "activeColors") as? [String] ?? []
        return stored.compactMap { ColorPalette(rawValue: $0) }
    }
}
```

---

### Использование в UI ([[SwiftUI]] и [[UIKit]])

#### SwiftUI Picker

```swift
import SwiftUI

enum FontSize: String, CaseIterable, Identifiable {
    case small = "Маленький"
    case medium = "Средний"
    case large = "Большой"
    
    var id: String { self.rawValue }
}

struct SettingsView: View {
    @State private var selectedSize = FontSize.medium
    
    var body: some View {
        Picker("Размер шрифта", selection: $selectedSize) {
            ForEach(FontSize.allCases) { size in
                Text(size.rawValue).tag(size)
            }
        }
    }
}
```

#### UIKit [[UISegmentedControl]]

```swift
import UIKit

enum SortOption: String, CaseIterable {
    case name = "По имени"
    case date = "По дате"
    case rating = "По рейтингу"
}

class ViewController: UIViewController {
    let segmentedControl = UISegmentedControl(items: SortOption.allCases.map { $0.rawValue })
    
    override func viewDidLoad() {
        super.viewDidLoad()
        segmentedControl.selectedSegmentIndex = 0
        segmentedControl.addTarget(self, action: #selector(segmentChanged), for: .valueChanged)
        view.addSubview(segmentedControl)
    }
    
    @objc func segmentChanged(_ sender: UISegmentedControl) {
        let selected = SortOption.allCases[sender.selectedSegmentIndex]
        print("Выбрано: \(selected)")
    }
}
```

---

### Работа с `allCases`

#### Доступ по индексу

```swift
enum Weekday: Int, CaseIterable {
    case mon = 1, tue, wed, thu, fri, sat, sun
}

let thirdDay = Weekday.allCases[2]  // wed
print(thirdDay.rawValue)  // 3
```

#### Поиск кейса

```swift
enum Action: String, CaseIterable {
    case start = "Старт"
    case stop = "Стоп"
    case pause = "Пауза"
}

func action(for title: String) -> Action? {
    return Action.allCases.first { $0.rawValue == title }
}

if let action = action(for: "Старт") {
    print(action)  // start
}
```

#### Использование с [[Identifiable]]

```swift
enum Tab: String, CaseIterable, Identifiable {
    case home, profile, settings
    
    var id: String { rawValue }
    
    var title: String {
        switch self {
        case .home: return "Главная"
        case .profile: return "Профиль"
        case .settings: return "Настройки"
        }
    }
}
```

---

### Расширения для `CaseIterable`

#### `randomElement()` для перечислений

```swift
extension CaseIterable where Self: Equatable {
    static func random() -> Self {
        return allCases.randomElement()!
    }
}

enum Coin: CaseIterable {
    case heads, tails
}

let flip = Coin.random()
print(flip)  // heads или tails
```

#### `count` как вычисляемое свойство

```swift
extension CaseIterable {
    static var count: Int {
        return allCases.count
    }
}

enum TrafficLight: CaseIterable {
    case red, yellow, green
}

print(TrafficLight.count)  // 3
```

#### `allCases` как [[Set Collection|Set]]

```swift
extension CaseIterable where Self: Hashable {
    static var allCasesSet: Set<Self> {
        return Set(allCases)
    }
}

enum Permission: String, CaseIterable {
    case read, write, delete
}

let permissionsSet = Permission.allCasesSet
print(permissionsSet.contains(.write))  // true
```

---

### Протоколы и дженерики с `CaseIterable`

```swift
func printAllCases<T: CaseIterable>(_ type: T.Type) where T: CustomStringConvertible {
    for item in T.allCases {
        print(item)
    }
}

enum Priority: String, CaseIterable, CustomStringConvertible {
    case low = "Низкий"
    case medium = "Средний"
    case high = "Высокий"
    
    var description: String { return rawValue }
}

printAllCases(Priority.self)
// Низкий
// Средний
// Высокий
```

---

### `CaseIterable` и [[Codable]]

Перечисления, соответствующие `CaseIterable` и `Codable`, автоматически кодируются и декодируются.

```swift
enum Status: String, CaseIterable, Codable {
    case active, inactive, pending
}

let status = Status.active
let encoded = try JSONEncoder().encode(status)
let decoded = try JSONDecoder().decode(Status.self, from: encoded)
print(decoded)  // active
```

---

### Особенности и ограничения

#### 1. **Порядок кейсов**

`allCases` возвращает кейсы в порядке их объявления в исходном коде.

```swift
enum Letters: CaseIterable {
    case b, a, c  // порядок: b, a, c
}

print(Letters.allCases)  // [b, a, c]
```

#### 2. **`@available` и `CaseIterable`**

Если кейс помечен `@available`, он не включается в `allCases` (если только весь тип не помечен той же версией).

```swift
@available(iOS 15.0, *)
enum ModernFeature: CaseIterable {
    case featureA, featureB
}

// allCases доступен только на iOS 15+
```

#### 3. **Производительность**

`allCases` создает массив каждый раз при обращении. Для часто используемых перечислений лучше сохранять результат в локальную переменную.

```swift
// ✅ Хорошо
let allDirections = Direction.allCases
for direction in allDirections { ... }

// ❌ Плохо (создаёт массив на каждой итерации)
for direction in Direction.allCases { ... }
```

---

### Лучшие практики

1.  **Используйте для конечных наборов значений** (дни недели, варианты фильтров).
2.  **Не используйте для перечислений с большим количеством кейсов** (например, 100+).
3.  **Сохраняйте `allCases` в локальную переменную**, если обращаетесь многократно.
4.  **Реализуйте `CaseIterable` вручную**, если перечисление имеет ассоциированные значения.
5.  **Комбинируйте с `Identifiable`** для SwiftUI списков и пикеров.

---

### Короткое правило

> **`CaseIterable`** позволяет перебрать все кейсы перечисления.  
> Компилятор синтезирует `allCases` для простых перечислений.  
> Используйте для выпадающих списков, пикеров и тестовых данных.

### Итог

**`CaseIterable`** в Swift:

1.  **Предоставляет свойство `allCases`** — массив всех кейсов перечисления.
2.  **Автоматически синтезируется** для перечислений без ассоциированных значений.
3.  **Может быть реализован вручную** для сложных случаев (indirect, ассоциированные значения).
4.  **Широко используется в UI** для заполнения `Picker`, `SegmentedControl`, `List`.
5.  **Недоступен для экзистенциальных типов** (`any CaseIterable`), только как ограничение дженерика.

Понимание `CaseIterable` упрощает работу с перечислениями и делает код более декларативным и безопасным .