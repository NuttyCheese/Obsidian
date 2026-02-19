**UITableViewDelegate** — это протокол в [[UIKit]], который отвечает за **взаимодействие пользователя** с таблицей и **управление её поведением** (в отличие от [[UITableViewDataSource]], который только даёт данные).

Он обрабатывает:

- нажатия и выделение строк  
- высоту строк, заголовков, футеров  
- подсветку / unhighlight  
- появление/исчезновение строк  
- скролл, swipe actions, editing mode и многое другое

### Обязательные методы?  
Нет обязательных.  
Все методы **опциональные** — реализуй только то, что нужно.

### Самые важные методы в 2026 году (реальный топ)

| Метод                                                      | Когда вызывается                         | Самый частый пример использования 2026               |
| ---------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------- |
| `tableView(_:didSelectRowAt:)`                             | Пользователь нажал на строку             | Открыть детальный экран, показать алерт              |
| `tableView(_:didDeselectRowAt:)`                           | Строка потеряла выделение                | Снять выделение, обновить UI                         |
| `tableView(_:heightForRowAt:)`                             | Запрос высоты строки                     | Динамическая высота (UITableView.automaticDimension) |
| `tableView(_:didHighlightRowAt:)` / `didUnhighlightRowAt:` | Строка нажата / отпущена (touch down/up) | Анимация нажатия (scale 0.97, alpha 0.8)             |
| `tableView(_:willDisplay:forRowAt:)`                       | Строка вот-вот появится на экране        | Prefetch изображений, анимации входа                 |
| `tableView(_:didEndDisplaying:forRowAt:)`                  | Строка ушла с экрана                     | Отменить загрузку изображений                        |
| `tableView(_:trailingSwipeActionsConfigurationForRowAt:)`  | Swipe справа (удалить, редактировать)    | .delete, .edit, custom actions                       |
| `tableView(_:leadingSwipeActionsConfigurationForRowAt:)`   | Swipe слева (архивировать, пометить)     | .archive, .flag                                      |
| `scrollViewDidScroll(_:)` (из UIScrollViewDelegate)        | Прокрутка таблицы                        | Parallax header, sticky sections                     |

### Самый современный паттерн 2026 (с [[UITableViewDiffableDataSource|DiffableDataSource]] + [[@MainActor]])

```swift
@MainActor
class TasksViewController: UIViewController {
    
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var dataSource: UITableViewDiffableDataSource<Int, Task>!
    private var tasks: [Task] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.delegate = self
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
        loadTasks()
    }
    
    private func configureDataSource() {
        dataSource = UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, task in
            let cell = tableView.dequeueReusableCell(withIdentifier: "TaskCell", for: indexPath) as! TaskCell
            cell.configure(with: task)
            return cell
        }
    }
    
    private func loadTasks() {
        // ... загрузка данных
        applySnapshot()
    }
    
    private func applySnapshot() {
        var snapshot = NSDiffableDataSourceSnapshot<Int, Task>()
        snapshot.appendSections([0])
        snapshot.appendItems(tasks)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}

extension TasksViewController: UITableViewDelegate {
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        
        guard let task = dataSource.itemIdentifier(for: indexPath) else { return }
        // Открыть детальный экран
        let detailVC = TaskDetailViewController(task: task)
        navigationController?.pushViewController(detailVC, animated: true)
    }
    
    func tableView(_ tableView: UITableView, 
                  trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
        let deleteAction = UIContextualAction(style: .destructive, title: "Delete") { _, _, completion in
            // Удаление задачи
            completion(true)
        }
        return UISwipeActionsConfiguration(actions: [deleteAction])
    }
    
    func tableView(_ tableView: UITableView, 
                  didHighlightRowAt indexPath: IndexPath) {
        if let cell = tableView.cellForRow(at: indexPath) {
            UIView.animate(withDuration: 0.15) {
                cell.transform = CGAffineTransform(scaleX: 0.97, y: 0.97)
                cell.contentView.alpha = 0.85
            }
        }
    }
    
    func tableView(_ tableView: UITableView, 
                  didUnhighlightRowAt indexPath: IndexPath) {
        if let cell = tableView.cellForRow(at: indexPath) {
            UIView.animate(withDuration: 0.15) {
                cell.transform = .identity
                cell.contentView.alpha = 1.0
            }
        }
    }
}
```

### Лучшие практики UITableViewDelegate в Swift 2026

- **[[Delegate]] = [[self]]** — чаще всего контроллер сам себе делегат  
- **didSelectRowAt** — основной метод для обработки нажатия (всегда deselectRow)  
- **didHighlight / didUnhighlight** — для красивой анимации нажатия (scale 0.97, alpha 0.85)  
- **trailingSwipeActionsConfigurationForRowAt** — для swipe-to-delete / edit (iOS 11+)  
- **heightForRowAt** — используй `UITableView.automaticDimension` для динамической высоты  
- **[[@MainActor]]** — весь контроллер или методы делегата — на главном акторе  
- **[[Swift]] 6 strict concurrency** — UITableViewDelegate методы вызываются на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «UITableViewDelegate — обработка выбора, swipe и анимации нажатия»

**Короткий девиз 2026**:
> UITableViewDelegate — это когда ты говоришь таблице:  
> «что делать, когда пользователь нажал строку, подсветил её, начал скроллить или строка появилась/исчезла».  
> В 2026 году основные методы — didSelect, didHighlight/didUnhighlight, swipe actions и heightForRowAt.  
> Всё остальное — в DiffableDataSource или Compositional Layout.
