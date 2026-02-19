**UITableViewController** — это специализированный подкласс **[[UIViewController]]** в [[UIKit]], который уже имеет встроенную поддержку **[[UITableView]]**.

Он предназначен для упрощения создания экранов, где **основной (а часто и единственный) контент** — это таблица.

### Основные особенности UITableViewController (актуально на 2026 год)

| Характеристика                                                                   | Описание                                                                          | Преимущества / Когда использовать в 2026       |
| -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------- |
| Встроенная `tableView`                                                           | Свойство `tableView: UITableView!` создаётся автоматически                        | Не нужно вручную создавать и добавлять таблицу |
| Автоматическое назначение dataSource и delegate                                  | `self` (контроллер) становится dataSource и [[delegate]] по умолчанию             | Меньше boilerplate кода                        |
| Поддержка **storyboard / [[xib]]**                                               | Можно подключить таблицу прямо в Interface Builder                                | Удобно для простых экранов                     |
| Поддержка **программного создания**                                              | Можно переопределить `loadView()` или использовать init                           | Полный контроль в коде                         |
| Полная совместимость с **[[UITableViewDiffableDataSource\|DiffableDataSource]]** | Работает с `UITableViewDiffableDataSource` без проблем                            | Стандарт 2026 года                             |
| Жизненный цикл                                                                   | Те же методы, что у UIViewController ([[viewDidLoad]], [[viewWillAppear]] и т.д.) | Никаких сюрпризов                              |
| Стили таблицы                                                                    | Можно передать в init(style:) — .plain, .grouped, .insetGrouped                   | Гибкость стиля                                 |

### Когда использовать UITableViewController в 2026 году

**Да, стоит использовать**, если:

- Экран **почти полностью состоит** из UITableView (список настроек, чат, контакты, история, уведомления)  
- Хочешь **минимум boilerplate** (не писать вручную addSubview, constraints)  
- Работаешь с **UIKit** (Storyboard, [[xib]], programmatic UIKit)  
- Нужна **быстрая разработка** простых списков

**Нет, лучше взять обычный UIViewController**, если:

- В экране **много другого UI** (header, footer, кнопки, поисковая строка, кастомный navigation bar)  
- Используешь **SwiftUI** — там `UITableViewController` не нужен  
- Хочешь **полный контроль** над иерархией вью (таблица — не главный view)  
- Используешь **[[TCA]] / [[MVVM (Model-View-ViewModel) Architecture|MVVM]]-[[Coordinator]] / [[VIPER Architecture|VIPER]]** — обычно предпочитают обычный контроллер + ViewModel

### Самый современный и рекомендуемый паттерн 2026 года

#### Программный подход (без Storyboard + DiffableDataSource)

```swift
final class TasksTableViewController: UITableViewController {
    
    // 1. Модель строки — обязательно Hashable!
    struct Task: Hashable {
        let id: UUID = UUID()
        let title: String
        let isCompleted: Bool
    }
    
    // 2. Типы для data source
    typealias DataSource = UITableViewDiffableDataSource<Int, Task>
    typealias Snapshot = NSDiffableDataSourceSnapshot<Int, Task>
    
    private var dataSource: DataSource!
    private var tasks: [Task] = []
    
    // 3. Инициализация с нужным стилем
    init() {
        super.init(style: .insetGrouped)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 4. Настройка таблицы (уже существует!)
        tableView.backgroundColor = .systemBackground
        tableView.register(TaskCell.self, forCellReuseIdentifier: "TaskCell")
        
        configureDataSource()
        loadInitialTasks()
    }
    
    private func configureDataSource() {
        // 5. Создаём data source один раз
        dataSource = UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, task in
            let cell = tableView.dequeueReusableCell(withIdentifier: "TaskCell", for: indexPath) as! TaskCell
            cell.configure(with: task)
            return cell
        }
    }
    
    private func loadInitialTasks() {
        // Пример данных
        tasks = [
            Task(title: "Buy milk", isCompleted: false),
            Task(title: "Call mom", isCompleted: true),
            Task(title: "Walk the dog", isCompleted: false)
        ]
        
        applySnapshot()
    }
    
    private func applySnapshot() {
        var snapshot = Snapshot()
        snapshot.appendSections([0])
        snapshot.appendItems(tasks)
        dataSource.apply(snapshot, animatingDifferences: false)
    }
    
    // 6. Обновление при изменении данных
    func toggleTaskCompletion(_ task: Task) {
        var snapshot = dataSource.snapshot()
        
        if let oldTaskIndex = snapshot.itemIdentifiers.firstIndex(of: task) {
            snapshot.deleteItems([task])
            
            let updatedTask = Task(title: task.title, isCompleted: !task.isCompleted)
            snapshot.appendItems([updatedTask])
            
            dataSource.apply(snapshot, animatingDifferences: true)
        }
    }
}
```

### Лучшие практики UITableViewController в Swift 2026

- **override init(style:)** — основной способ создания  
- **Не забывай super.init(style:)** — иначе таблица не инициализируется  
- **tableView** — уже существует и настроено, используй его напрямую  
- **[[@MainActor]]** — весь контроллер — на главном акторе  
- **[[UITableViewDiffableDataSource|DiffableDataSource]]** — **обязательно** для новых проектов (анимации, производительность)  
- **Swift 6 strict concurrency** — UITableViewController полностью безопасен (@MainActor)  
- **Документируйте** — пиши комментарий «UITableViewController — экран со списком задач»

**Короткий девиз 2026**:
> UITableViewController — это когда тебе нужен **готовый контроллер с таблицей внутри**, минимум boilerplate и максимальная скорость разработки.  
> В 2026 году это **отличный выбор** для экранов, где 90%+ контента — это UITableView.  
> Если экран сложный (header, footer, кнопки, поиск) — лучше обычный UIViewController + tableView как subview.
