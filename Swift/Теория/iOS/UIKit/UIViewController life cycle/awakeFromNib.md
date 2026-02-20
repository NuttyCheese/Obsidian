**`awakeFromNib()`** — это метод жизненного цикла в UIKit, который вызывается **после** того, как объект (обычно `UIView` или `UIViewController`) был загружен из nib-файла (Storyboard или .xib).

### Когда и зачем вызывается awakeFromNib()

| Событие                              | Порядок вызова (примерно)                          | awakeFromNib вызывается? | Что уже доступно к моменту вызова |
|--------------------------------------|----------------------------------------------------|---------------------------|-----------------------------------|
| init(coder:)                         | Первый (при загрузке из nib/Storyboard)            | Нет                       | Ничего (outlets ещё nil)          |
| awakeFromNib()                       | Сразу после полной загрузки nib и установки всех outlets | **Да**                    | Все IBOutlet-свойства уже не nil  |
| viewDidLoad() (для контроллера)      | После awakeFromNib                                 | —                         | —                                 |
| layoutSubviews() / viewDidLayoutSubviews() | Позже (при изменении размеров)                     | —                         | —                                 |

**Ключевой момент 2026 года**:
> awakeFromNib() — это **последний безопасный момент**, когда вы можете обращаться к **IBOutlet** и **IBAction** без риска получить nil.

### Самые популярные и правильные сценарии использования awakeFromNib() в 2026 году

#### 1. Настройка UI-элементов, которые пришли из nib (самый частый случай)

```swift
class CustomButton: UIButton {
    
    @IBOutlet private weak var iconImageView: UIImageView!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        // Теперь iconImageView точно не nil
        iconImageView.tintColor = .systemBlue
        iconImageView.contentMode = .scaleAspectFit
        
        // Настраиваем кнопку
        layer.cornerRadius = 12
        backgroundColor = .systemBackground
        setTitleColor(.systemBlue, for: .normal)
    }
}
```

#### 2. Инициализация состояния, зависящего от outlets

```swift
class ProfileHeaderView: UIView {
    
    @IBOutlet private weak var avatarImageView: UIImageView!
    @IBOutlet private weak var nameLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        avatarImageView.layer.cornerRadius = avatarImageView.bounds.height / 2
        avatarImageView.clipsToBounds = true
        
        nameLabel.font = .preferredFont(forTextStyle: .headline)
    }
}
```

#### 3. В UIViewController (очень редко, но допустимо)

```swift
class SettingsViewController: UIViewController {
    
    @IBOutlet private weak var tableView: UITableView!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        // Редкий случай: настройка, которую нужно сделать до viewDidLoad
        tableView.backgroundColor = .systemGroupedBackground
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Основная логика здесь
    }
}
```

### Чего **НИКОГДА** нельзя делать в awakeFromNib() в 2026 году

| Запрещённое действие                              | Почему нельзя делать                                   | Куда перенести |
|---------------------------------------------------|--------------------------------------------------------|----------------|
| Обращение к frame / bounds / layout               | Размеры ещё не установлены (layout не выполнен)        | layoutSubviews() или viewDidLayoutSubviews() |
| Вызов методов, зависящих от superview             | superview может быть ещё nil                           | didMoveToSuperview() |
| Тяжёлые вычисления или сетевые запросы            | Метод вызывается синхронно при загрузке nib            | viewDidLoad() или позже |
| Изменение иерархии view (addSubview и т.п.)       | Может нарушить загрузку из nib                         | viewDidLoad() |

### Лучшие практики awakeFromNib() в Swift 2026

- **Используйте** awakeFromNib() **только** для:
  - настройки свойств IBOutlet (цвета, шрифты, cornerRadius, clipsToBounds и т.п.)
  - установки начального состояния UI-элементов из nib
  - подготовки ячеек/вью, которые загружаются из .xib
- **Никогда** не используйте для:
  - layout-логики (размеры, позиционирование)
  - добавления/удаления subviews
  - сетевых запросов, тяжёлых вычислений
- **Вызывайте** `super.awakeFromNib()` — обязательно (особенно в подклассах UIView / UIViewController)
- **В SwiftUI** — awakeFromNib() **не существует** (UIHostingController и UIViewRepresentable используют другие жизненные циклы)
- **Для Storyboard** — awakeFromNib() вызывается **один раз** при первой загрузке
- **Документируйте** — пишите комментарий «awakeFromNib — настройка UI-элементов, загруженных из nib (outlets уже доступны)»

**Короткий итог 2026**:
> `awakeFromNib()` — это **последний безопасный момент** после загрузки из .xib / Storyboard, когда все IBOutlet уже не nil.  
> В 2026 году:  
> - используйте **только** для настройки свойств UI-элементов (цвета, шрифты, cornerRadius и т.п.)  
> - **никогда** не делайте layout, добавление subviews, сетевые запросы  
> - вызывайте `super.awakeFromNib()`  
> Это **классический** и **очень полезный** метод для работы с nib-файлами и кастомными UIView.

Удачи с чистой и надёжной инициализацией UI из nib в твоём проекте! 🧩