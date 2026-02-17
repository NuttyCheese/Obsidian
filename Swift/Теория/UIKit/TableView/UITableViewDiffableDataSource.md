**UITableViewDiffableDataSource** — это современный класс из UIKit (с iOS 13, 2019), который радикально упрощает работу с данными в таблицах.

Он заменяет старый подход с `UITableViewDataSource` + `reloadData()` / `performBatchUpdates()` на **снимки данных** (snapshots) и **автоматический diff**.

### Почему в 2025–2026 годах почти все используют именно DiffableDataSource

| Сравнение                              | Классический UITableViewDataSource | UITableViewDiffableDataSource (2026) | Победитель |
|----------------------------------------|-------------------------------------|---------------------------------------|------------|
| Управление обновлениями                | Ручное: reloadData(), insert/delete вручную | Автоматический diff + анимации        | **Diffable** |
| Анимации                               | Нужно писать performBatchUpdates     | Встроенные, плавные, безопасные       | **Diffable** |
| Код и читаемость                       | Много boilerplate                    | Минимум кода, декларативно            | **Diffable** |
| Производительность на больших списках | Хорошая, но легко ошибиться          | Лучше (diff считает минимум изменений) | **Diffable** |
| Обработка состояний (loading, empty, error) | Ручная логика                        | Легко через разные snapshots          | **Diffable** |
| Swift Concurrency                      | Требует осторожности                 | Полностью безопасен (+ Sendable)      | **Diffable** |
| Поддержка Apple                        | Поддерживается, но без новых фич     | Активно развивается                   | **Diffable** |

### Самый современный и рекомендуемый паттерн 2026 года

```swift
import UIKit

final class TasksViewController: UIViewController {
    
    // 1. Определяем секции (enum — самый удобный способ)
    enum Section: Hashable {
        case active
        case completed
    }
    
    // 2. Модель строки — обязательно Hashable!
    struct Task: Hashable {
        let id: UUID = UUID()
        let title: String
        let isCompleted: Bool
        let priority: Int
    }
    
    // 3. Типы для data source
    typealias DataSource = UITableViewDiffableDataSource<Section, Task>
    typealias Snapshot = NSDiffableDataSourceSnapshot<Section, Task>
    
    private var dataSource: DataSource!
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var tasks: [Task] = []  // ваш источник данных
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.register(TaskCell.self, forCellReuseIdentifier: "TaskCell")
        
        view.addSubview(tableView)
        tableView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        configureDataSource()
        loadInitialTasks()
    }
    
    private func configureDataSource() {
        // 4. Создаём data source один раз
        dataSource = UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, task in
            let cell = tableView.dequeueReusableCell(withIdentifier: "TaskCell", for: indexPath) as! TaskCell
            cell.configure(with: task)
            return cell
        }
        
        // (Опционально) кастомизация заголовков секций
        dataSource.supplementaryViewProvider = { tableView, kind, indexPath in
            guard kind == UITableView.sectionHeader else { return nil }
            let header = tableView.dequeueReusableHeaderFooterView(withIdentifier: "header") as! UITableViewHeaderFooterView
            let section = self.dataSource.snapshot().sectionIdentifiers[indexPath.section]
            header.textLabel?.text = section == .active ? "Active Tasks" : "Completed Tasks"
            return header
        }
    }
    
    private func loadInitialTasks() {
        // 5. Создаём начальный снимок
        var snapshot = Snapshot()
        snapshot.appendSections([.active, .completed])
        
        // Пример данных
        let active = [
            Task(title: "Buy milk", isCompleted: false, priority: 1),
            Task(title: "Call mom", isCompleted: false, priority: 2)
        ]
        let completed = [
            Task(title: "Walk the dog", isCompleted: true, priority: 0)
        ]
        
        snapshot.appendItems(active, toSection: .active)
        snapshot.appendItems(completed, toSection: .completed)
        
        dataSource.apply(snapshot, animatingDifferences: false)
    }
    
    // 6. Как обновлять данные (самое важное!)
    func toggleTaskCompletion(_ task: Task) {
        var snapshot = dataSource.snapshot()
        
        // Находим и удаляем из старой секции
        if let oldSection = snapshot.sectionIdentifier(containingItem: task) {
            snapshot.deleteItems([task])
        }
        
        // Создаём обновлённую задачу
        let updatedTask = Task(title: task.title, isCompleted: !task.isCompleted, priority: task.priority)
        
        // Добавляем в новую секцию
        let newSection: Section = updatedTask.isCompleted ? .completed : .active
        snapshot.appendItems([updatedTask], toSection: newSection)
        
        // Применяем с анимацией
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

### Короткий чек-лист «Что нужно сделать, чтобы DiffableDataSource заработал»

1. Определить `Section` (enum Hashable)  
2. Определить модель строки (struct Hashable)  
3. Создать `typealias DataSource` и `Snapshot`  
4. Зарегистрировать ячейки  
5. Создать dataSource с замыканием cellProvider  
6. Создать snapshot, добавить секции и элементы  
7. Применить snapshot через `apply(_:animatingDifferences:)`

### Короткий девиз 2026

> UITableViewDiffableDataSource — это когда ты хочешь **автоматические анимации**, **меньше багов** и **меньше кода** при обновлении таблицы.  
> В 2026 году это **единственный рекомендуемый** способ управлять данными в UITableView.  
> Забудь reloadData() и performBatchUpdates() — используй snapshots и apply().

Удачи с плавными, красивыми и современными списками! 📜