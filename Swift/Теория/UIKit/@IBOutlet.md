**@IBOutlet** — это специальный атрибут в Swift, который позволяет **связать переменную в коде** с UI-элементом, созданным в **Interface Builder** (Storyboard или XIB).

Проще говоря:  
`@IBOutlet` = «переменная в коде, которая автоматически заполняется ссылкой на элемент интерфейса из IB».

### Зачем нужен @IBOutlet (2026 актуально)

- Доступ к UI-элементам из кода **без ручного создания**  
- Удобно менять свойства (текст, цвет, скрытие, изображение) прямо в `viewDidLoad`  
- Работает только с элементами, которые наследуются от `NSObject` (UILabel, UIButton, UIImageView, UISlider, UITableView, UICollectionView и т.д.)  
- Под капотом использует **@objc** — runtime Objective-C устанавливает связь  
- Предотвращает retain cycle через **weak** ссылку

### Основной синтаксис (самый частый и правильный в 2026)

```swift
@IBOutlet weak var titleLabel: UILabel!
@IBOutlet weak var loginButton: UIButton!
@IBOutlet weak var avatarImageView: UIImageView!
```

- **weak** — обязательно (чтобы не было retain cycle)  
- **!** (implicitly unwrapped optional) — стандарт, потому что связь устанавливается после `viewDidLoad`  
- Можно использовать **?** (обычный optional), но тогда нужно разворачивать через guard/optional chaining

### Самые частые примеры использования @IBOutlet

#### 1. UILabel — изменение текста

```swift
@IBOutlet weak var greetingLabel: UILabel!

override func viewDidLoad() {
    super.viewDidLoad()
    greetingLabel.text = "Привет, \(userName)!"
    greetingLabel.textColor = .systemBlue
}
```

#### 2. UIButton — настройка внешнего вида

```swift
@IBOutlet weak var submitButton: UIButton!

override func viewDidLoad() {
    super.viewDidLoad()
    submitButton.setTitle("Отправить", for: .normal)
    submitButton.backgroundColor = .systemGreen
    submitButton.layer.cornerRadius = 12
}
```

#### 3. UIImageView — загрузка изображения

```swift
@IBOutlet weak var profileImageView: UIImageView!

override func viewDidLoad() {
    super.viewDidLoad()
    profileImageView.image = UIImage(named: "avatar")
    profileImageView.layer.cornerRadius = profileImageView.bounds.width / 2
    profileImageView.clipsToBounds = true
}
```

#### 4. UITableView / UICollectionView — настройка таблицы

```swift
@IBOutlet weak var tableView: UITableView!

override func viewDidLoad() {
    super.viewDidLoad()
    tableView.dataSource = self
    tableView.delegate = self
    tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
}
```

#### 5. IBOutlet Collection — группа элементов

```swift
@IBOutlet var buttons: [UIButton]!

override func viewDidLoad() {
    super.viewDidLoad()
    for (index, button) in buttons.enumerated() {
        button.setTitle("Button \(index + 1)", for: .normal)
        button.tag = index + 1
    }
}
```

### Как правильно подключать @IBOutlet в Interface Builder

1. Открой Storyboard / XIB  
2. Выдели нужный UI-элемент (UILabel, UIButton и т.д.)  
3. Ctrl+Drag (или правой кнопкой → drag) от элемента к классу контроллера в коде  
4. Выбери **Connection: Outlet**  
5. Укажи имя переменной (например, `titleLabel`)  
6. Xcode автоматически создаст строку `@IBOutlet weak var titleLabel: UILabel!`

### Лучшие практики @IBOutlet в Swift 2026

- **weak** — всегда (retain cycle)  
- **!** — стандарт для implicitly unwrapped (если уверен, что связь будет установлена)  
- **?** — если элемент может отсутствовать (например, в разных конфигурациях экрана)  
- **IBOutlet Collection** — для группы кнопок/лейблов/вью  
- **@MainActor** — весь контроллер — на главном акторе  
- **Swift 6 strict concurrency** — @IBOutlet полностью безопасен  
- **Документируйте** — пиши комментарий «@IBOutlet — метка приветствия из Storyboard»

**Короткий девиз 2026**:
> @IBOutlet — это когда ты хочешь **управлять элементом из Interface Builder** прямо из кода: менять текст, цвет, изображение, скрывать/показывать.  
> В 2026 году это **самый удобный** способ работы с UI в UIKit-проектах со Storyboard/XIB.  
> Всегда weak + ! и подключай через Ctrl+Drag.

Удачи с быстрым и удобным доступом к UI-элементам в Swift! 🎛️