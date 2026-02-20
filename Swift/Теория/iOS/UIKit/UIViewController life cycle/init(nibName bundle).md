**`init(nibName:bundle:)`** — это **designated инициализатор** класса [[UIViewController]] в [[UIKit]], который используется для программной загрузки контроллера из **.[[xib]]**-файла (или nib-файла).

Это один из самых распространённых и классических способов создания контроллера представления из отдельного nib-файла (в отличие от Storyboard или чисто кодового создания).

### Когда и зачем используется init(nibName:bundle:)

| Ситуация                                  | Почему именно этот инициализатор                    | Альтернатива                                              |
| ----------------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| Контроллер имеет отдельный .xib-файл      | Самый чистый и рекомендуемый способ загрузки из nib | `init()` + `loadViewFromNib()` вручную                    |
| Нужна точная загрузка конкретного nib     | Можно явно указать имя nib и бандл                  | Storyboard → `instantiateViewController(withIdentifier:)` |
| Поддержка старых проектов (до [[iOS]] 13) | До появления Storyboard это был основной способ     | —                                                         |
| Локализация через разные nib-файлы        | Разные nib для разных языков/регионов               | Локализация строк, но не nib                              |

### Полная сигнатура

```swift
init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?)
```

- `nibNameOrNil`: имя .xib-файла **без расширения** `.xib`  
  - Если передать `nil` → ищет файл с именем класса контроллера (например, `MyViewController.xib`)
- `nibBundleOrNil`: бандл, в котором лежит nib  
  - Если `nil` → берётся `Bundle(for: type(of: self))` — бандл текущего класса

### Самые популярные и рекомендуемые паттерны в 2026 году

#### 1. Классический вариант (самый частый)

```swift
class ProfileViewController: UIViewController {
    
    // Явно указываем имя nib и бандл
    init() {
        super.init(nibName: "ProfileViewController", bundle: nil)
        // или
        // super.init(nibName: nil, bundle: nil) — ищет ProfileViewController.xib
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // outlets уже доступны
    }
}
```

#### 2. С передачей бандла (очень полезно в [[SPM]] / модульных проектах)

```swift
class SettingsViewController: UIViewController {
    
    init() {
        let bundle = Bundle.module  // для Swift Package Manager
        // или Bundle(for: SettingsViewController.self)
        
        super.init(nibName: "SettingsView", bundle: bundle)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

#### 3. Полный пример с кастомным nib-именем

```swift
class ReusableHeaderViewController: UIViewController {
    
    static func instantiate() -> ReusableHeaderViewController {
        return ReusableHeaderViewController(nibName: "HeaderView", bundle: nil)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // настройка UI из nib
    }
}
```

### Чего **НИКОГДА** нельзя делать в init(nibName:bundle:)

| Запрещённое действие                              | Почему нельзя делать                                   | Куда перенести |
|---------------------------------------------------|--------------------------------------------------------|----------------|
| Обращение к IBOutlet / IBAction                   | Они ещё nil — загрузка nib не завершена                | `viewDidLoad()` или `awakeFromNib()` |
| Доступ к `view`, `view.bounds`, `view.frame`      | `view` ещё не создан (lazy-loading)                    | `viewDidLoad()` или `viewDidAppear()` |
| Добавление subviews вручную                       | Может сломать загрузку из nib                          | `viewDidLoad()` |
| Тяжёлые операции (сеть, диск, вычисления)         | Задерживает создание контроллера                       | `viewDidLoad()` или позже |

### Лучшие практики init(nibName:bundle:) в Swift 2026

- **Делайте** инициализатор **convenience**, если возможно:

  ```swift
  convenience init() {
      self.init(nibName: "MyViewController", bundle: nil)
  }
  ```

- **Всегда** реализуйте **required init?(coder:)** с `fatalError` (если контроллер не поддерживает Storyboard)

  ```swift
  required init?(coder: NSCoder) {
      fatalError("init(coder:) has not been implemented")
  }
  ```

- **Для SPM** — используйте `Bundle.module` или `Bundle(for: Self.self)`
- **Для локализации** — создавайте отдельные nib-файлы для разных языков (Base.lproj, ru.lproj и т.д.)
- **В [[SwiftUI]]** — `init(nibName:bundle:)` **не используется** (вместо него `UIViewControllerRepresentable`)
- **Документируйте** — пишите комментарий «init(nibName:bundle:) — загрузка контроллера из отдельного .xib-файла»

**Короткий итог 2026**:
> `init(nibName:bundle:)` — это **designated инициализатор** для контроллеров, загружаемых из .xib-файла.  
> В 2026 году:  
> - вызывайте `super.init(nibName:bundle:)`  
> - **нельзя** обращаться к `view`, `IBOutlet`, делать layout  
> - всю логику — в `viewDidLoad()` / `awakeFromNib()`  
> - если контроллер не поддерживает Storyboard — `fatalError` в `init(coder:)`  
> Это **классический** и **надёжный** способ работы с nib-файлами в UIKit.
