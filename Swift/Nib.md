**Nib** (или **.xib**) — это старый, но всё ещё активно используемый формат файла интерфейса в [[UIKit]] (с iOS 3.0 по настоящее время, 2026 год).

Это **бинарный сериализованный архив** объектов UIKit ([[UIView]], [[UILabel]], [[UIButton]], [[UITableViewCell]] и т.д.), созданный в Interface Builder.

Основное назначение в 2026 году:
- создание **кастомных ячеек** для [[UITableView]] / [[UICollectionView]]
- выделение повторяющихся UI-компонентов (баннеры, карточки, поповеры, кастомные вью)
- поддержка legacy-проектов и команд, которые всё ещё используют Storyboard/XIB

### Сравнение Nib vs Storyboard vs SwiftUI (2026 реальность)

| Характеристика                  | Nib (.xib)                              | Storyboard                              | SwiftUI (View)                          | Победитель в 2026 |
|---------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|-------------------|
| Формат                          | Один файл = один объект (чаще UIView)   | Один файл = много экранов               | Код (struct View)                       | SwiftUI           |
| Поддержка кастомных ячеек       | Отлично                                 | Хорошо                                  | LazyVGrid / List — идеально             | Nib / SwiftUI     |
| Размер файла                    | Очень маленький                         | Часто огромный (много экранов)          | Нет файла — только код                  | Nib               |
| Merge-конфликты в git           | Почти никогда                           | Часто (Storyboard — ад для merge)       | Нет (код)                               | Nib / SwiftUI     |
| Производительность загрузки     | Очень быстрая                           | Медленнее (парсинг большого файла)      | Самая быстрая (компиляция)              | SwiftUI           |
| Поддержка в новых проектах      | Только для ячеек и компонентов          | Умирает (Apple не развивает)            | Основной выбор                          | SwiftUI           |
| Легко переиспользовать          | Да (UINib.instantiate)                  | Сложно (storyboard ID)                  | Очень легко (struct)                    | SwiftUI / Nib     |

### Как правильно использовать Nib в 2026 году (рекомендуемый паттерн)

#### 1. Кастомная ячейка для UITableView / UICollectionView

```swift
// CustomCell.xib → File's Owner = CustomCell.self
final class CustomCell: UITableViewCell {
    
    @IBOutlet private weak var titleLabel: UILabel!
    @IBOutlet private weak var subtitleLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        // Здесь можно делать настройки, которые не зависят от данных
        titleLabel.font = .preferredFont(forTextStyle: .headline)
    }
    
    func configure(title: String, subtitle: String) {
        titleLabel.text = title
        subtitleLabel.text = subtitle
    }
}

// Регистрация один раз (обычно в viewDidLoad)
tableView.register(UINib(nibName: "CustomCell", bundle: nil), 
                   forCellReuseIdentifier: "CustomCell")

// Использование
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "CustomCell", for: indexPath) as! CustomCell
    cell.configure(title: "Задача \(indexPath.row)", subtitle: "Описание")
    return cell
}
```

#### 2. Переиспользуемый кастомный UIView (баннер, карточка, поповер)

```swift
final class BannerView: UIView {
    
    @IBOutlet private weak var titleLabel: UILabel!
    @IBOutlet private weak var actionButton: UIButton!
    
    // Самый правильный способ загрузки Nib
    override init(frame: CGRect) {
        super.init(frame: frame)
        commonInit()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        commonInit()
    }
    
    private func commonInit() {
        let nib = UINib(nibName: "BannerView", bundle: nil)
        guard let view = nib.instantiate(withOwner: self, options: nil).first as? UIView else {
            fatalError("Не удалось загрузить BannerView.xib")
        }
        
        // Важно: добавляем в self, а не в contentView
        addSubview(view)
        view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            view.topAnchor.constraint(equalTo: topAnchor),
            view.leadingAnchor.constraint(equalTo: leadingAnchor),
            view.trailingAnchor.constraint(equalTo: trailingAnchor),
            view.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }
    
    func configure(title: String, buttonTitle: String, action: @escaping () -> Void) {
        titleLabel.text = title
        actionButton.setTitle(buttonTitle, for: .normal)
        actionButton.addAction(UIAction { _ in action() }, for: .touchUpInside)
    }
}
```

### Лучшие практики использования Nib в Swift 2026

- **Только для ячеек и переиспользуемых компонентов** — не делай весь экран в Nib  
- **File's Owner** — всегда устанавливай в класс (CustomCell.self, BannerView.self)  
- **awakeFromNib()** — идеальное место для базовой настройки (шрифты, цвета, cornerRadius)  
- **commonInit()** — стандартный паттерн для UIView из Nib (init(frame) + init(coder))  
- **UINib(nibName:bundle:)** — загружай один раз (лучше кэшировать в static let)  
- **Swift 6 strict concurrency** — Nib загрузка безопасна, но UI-обновления — @MainActor  
- **Документируйте** — пиши комментарий «CustomCell.xib — кастомная ячейка с IBOutlet»

**Короткий девиз 2026**:
> Nib (.xib) в 2026 году — это **узкоспециализированный инструмент** для кастомных ячеек таблиц/коллекций и переиспользуемых UIView-компонентов.  
> Не используй для целых экранов — это устарело.  
> SwiftUI → основной выбор для нового UI, Nib → только там, где нужен максимальный контроль над ячейкой без кода.
