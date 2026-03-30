**`didSet`** — это **наблюдатель свойства** (property observer) в [[Swift]], который автоматически вызывается **сразу после** того, как свойство получило **новое значение** (после присваивания).

Это один из самых удобных и часто используемых инструментов для реакции на изменения свойств — особенно в [[UIKit]], SwiftUI (через `@Published`) и ViewModel’ах.

### 1. Ключевые правила didSet (запомни навсегда)

| Правило                                                | Что значит                                                                   | Пример / Последствие                                                       |
| ------------------------------------------------------ | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Работает **только** с **хранимыми свойствами** (`var`) | Не работает с [[let]], computed properties ([[getter]]/[[setter]]), [[lazy]] | `var count = 0 { didSet { ... } }` — OK                                    |
| **Не вызывается** при **инициализации** свойства       | При [[init]] или объявлении значения [[didSet]] не срабатывает               | `var score = 0 { didSet { print("changed") } }` — при создании не печатает |
| Вызывается **после** присваивания нового значения      | `oldValue` — это значение **до** изменения                                   | `score = 10` → `oldValue` = предыдущее, `score` = 10                       |
| Можно использовать **несколько наблюдателей**          | `willSet` + `didSet` в одном свойстве                                        | `willSet` → до, `didSet` → после                                           |
| Доступ к `oldValue` и `newValue`                       | `oldValue` — встроенная переменная в `didSet`                                | `didSet { print("from \(oldValue) to \(score)") }`                         |
| Можно **изменять само свойство** внутри `didSet`       | Но это вызовет **ещё один** `didSet` (рекурсия!)                             | Опасно — легко получить бесконечный цикл                                   |

### 2. Самый рекомендуемый паттерн 2026 года (золотой стандарт)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    
    @Published private(set) var username: String = "" {
        didSet {
            // Автоматическое обновление UI или валидация
            updateGreeting()
            validateUsername()
        }
    }
    
    @Published var greeting: String = "Привет, гость!"
    
    private func updateGreeting() {
        greeting = username.isEmpty ? "Привет, гость!" : "Привет, \(username)!"
    }
    
    private func validateUsername() {
        if username.count > 20 {
            username = String(username.prefix(20))  // обрезаем
            // или показать ошибку
        }
    }
}
```

### 3. Полный разбор всех популярных сценариев didSet

#### Сценарий 1: Автоматическое обновление UI ([[UIKit]])

```swift
class ProfileViewController: UIViewController {
    
    @IBOutlet private weak var nameLabel: UILabel!
    
    var userName: String = "" {
        didSet {
            nameLabel.text = userName.isEmpty ? "Гость" : userName
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        userName = "Алексей"  // → label обновится
    }
}
```

#### Сценарий 2: Валидация и коррекция значения

```swift
class Settings {
    var volume: Double = 0.5 {
        didSet {
            // Ограничиваем диапазон 0...1
            if volume < 0 {
                volume = 0
            } else if volume > 1 {
                volume = 1
            }
            
            applyVolumeToPlayer(volume)
        }
    }
}
```

#### Сценарий 3: Логирование и отладка

```swift
class Counter {
    var value = 0 {
        didSet {
            print("Counter changed: \(oldValue) → \(value)")
        }
    }
}
```

#### Сценарий 4: didSet + [[willSet]] вместе (полный контроль)

```swift
var temperature: Double = 20.0 {
    willSet {
        print("Температура будет изменена на \(newValue)°C")
    }
    didSet {
        if temperature > 30 {
            print("Внимание: жарко! Было \(oldValue)°C")
        }
    }
}
```

### 4. Важные ловушки и антипаттерны (2026)

- **Рекурсия в didSet**  
  ```swift
  var score = 0 {
      didSet { score += 1 }  // Бесконечный цикл!
  }
  ```

- **didSet в computed property** — **запрещено** компилятором  
  ```swift
  var fullName: String {
      get { firstName + " " + lastName }
      set { /* ... */ }
      didSet { }  // Ошибка компиляции
  }
  ```

- **Забыли [weak self] в замыканиях внутри didSet** → retain cycle  
- **didSet вызывается даже при присваивании того же значения**  
  ```swift
  score = score  // didSet всё равно сработает!
  ```

### 5. Лучшие практики didSet в Swift 2026

- **Используй didSet для побочных эффектов**: UI-обновление, логи, валидация, запуск анимаций  
- **Не делай тяжёлую работу** в didSet — это может замедлить интерфейс  
- **В [[SwiftUI]]** — предпочитай `@Published` + `.onChange` или `didSet` внутри `@Published`  
- **В [[Combine]]** — используй `.sink` или `.assign` вместо didSet  
- **[[Swift]] 6 strict concurrency** — didSet выполняется на том же акторе, что и присваивание  
- **Документируйте** — пиши комментарий «didSet — автоматическое обновление лейбла при изменении имени»

**Короткий девиз 2026**:
> `didSet` — это «после того, как свойство изменилось — сделай вот это».  
> В 2026 году используй его для:  
> - автообновления UI  
> - валидации и коррекции значения  
> - логирования изменений  
> - запуска побочных эффектов  
> Но помни: **не вызывается при инициализации**, **может быть рекурсивным**, **работает только с var**.
