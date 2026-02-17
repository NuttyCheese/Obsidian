**UITableViewCell** — это базовый класс ячейки для **[[UITableView]]** в [[UIKit]].

Он отвечает за отображение **одной строки** таблицы и является контейнером для вашего контента (текст, картинки, кнопки, кастомные вью и т.д.).

В 2026 году это по-прежнему **единственный правильный** способ создавать строки таблицы в UIKit.

### Зачем нужен именно UITableViewCell (а не просто UIView)

- Автоматически поддерживает **переиспользование** (reuse) → огромная экономия памяти на длинных списках  
- Имеет встроенный `contentView` — **всегда** добавляй UI именно туда (не в self)  
- Поддерживает **выделение**, **выбранное состояние**, **editing mode**, **swipe actions**, **accessibility**  
- Работает с **[[UITableViewDiffableDataSource]]** и автоматической высотой  
- Имеет встроенные стили (.default, .subtitle, .value1, .value2) и аксессуары (.disclosureIndicator, .checkmark и т.д.)

### Самый современный способ создания ячейки в 2026 году

#### Программная кастомная ячейка (рекомендуемый подход)

```swift
final class TaskCell: UITableViewCell {
    
    // UI-элементы (лениво, чтобы не создавать, пока не понадобятся)
    private lazy var titleLabel: UILabel = {
        let label = UILabel()
        label.font = .preferredFont(forTextStyle: .body)
        label.textColor = .label
        return label
    }()
    
    private lazy var subtitleLabel: UILabel = {
        let label = UILabel()
        label.font = .preferredFont(forTextStyle: .caption1)
        label.textColor = .secondaryLabel
        return label
    }()
    
    private lazy var checkmarkImageView: UIImageView = {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFit
        iv.tintColor = .systemGreen
        return iv
    }()
    
    // Конфигурация один раз при создании ячейки
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        // Важно: всё добавляем в contentView!
        contentView.addSubview(titleLabel)
        contentView.addSubview(subtitleLabel)
        contentView.addSubview(checkmarkImageView)
        
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        subtitleLabel.translatesAutoresizingMaskIntoConstraints = false
        checkmarkImageView.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            titleLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: checkmarkImageView.leadingAnchor, constant: -8),
            
            subtitleLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 4),
            subtitleLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            subtitleLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor),
            subtitleLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12),
            
            checkmarkImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            checkmarkImageView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            checkmarkImageView.widthAnchor.constraint(equalToConstant: 24),
            checkmarkImageView.heightAnchor.constraint(equalToConstant: 24)
        ])
    }
    
    // Основной метод конфигурации (вызывается в cellForRowAt)
    func configure(with task: Task) {
        titleLabel.text = task.title
        subtitleLabel.text = task.dueDate?.formatted(date: .abbreviated, time: .shortened)
        checkmarkImageView.image = task.isCompleted ? UIImage(systemName: "checkmark.circle.fill") : nil
        checkmarkImageView.isHidden = !task.isCompleted
        
        // Цвет текста в зависимости от состояния
        titleLabel.textColor = task.isCompleted ? .secondaryLabel : .label
    }
    
    // Обязательно сбрасываем состояние при переиспользовании
    override func prepareForReuse() {
        super.prepareForReuse()
        titleLabel.text = nil
        subtitleLabel.text = nil
        checkmarkImageView.image = nil
        checkmarkImageView.isHidden = true
    }
}
```

### Как зарегистрировать и использовать ячейку

```swift
// В viewDidLoad или где создаётся таблица
tableView.register(TaskCell.self, forCellReuseIdentifier: "TaskCell")

// В data source
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "TaskCell", for: indexPath) as! TaskCell
    let task = tasks[indexPath.row]
    cell.configure(with: task)
    return cell
}
```

### Лучшие практики UITableViewCell в Swift 2026

- **[[final]] [[class]]** — экономия [[vtable]], меньше overhead  
- **[[lazy]] [[var]]** — UI создаётся только при первом обращении  
- **contentView** — **всегда** добавляй subviews именно туда  
- **configure(with:)** — отдельный метод для заполнения данными  
- **prepareForReuse()** — обязательно сбрасывай состояние (изображения, текст, цвета, alpha)  
- **[[@MainActor]]** — если ячейка обновляет UI из async-кода  
- **[[Swift]] 6 strict concurrency** — UITableViewCell не [[Sendable]] → избегай передачи в Task без @MainActor  
- **Документируйте** — пиши комментарий «UITableViewCell — кастомная ячейка задачи с ленивой инициализацией»

**Короткий девиз 2026**:
> UITableViewCell — это **контейнер для одной строки таблицы**.  
> Создаёшь подкласс, добавляешь UI в contentView, настраиваешь в configure(with:), регистрируешь один раз.  
> В 2026 году — всегда programmatic + final + lazy + configure-метод + prepareForReuse.
