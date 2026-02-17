**UITableViewDataSource** — это протокол в UIKit, который отвечает за **предоставление данных** таблице (`UITableView`).

Он говорит таблице:
- сколько секций
- сколько строк в каждой секции
- какую ячейку показать для каждой строки

Это **обязательный** протокол — без него таблица просто не отобразит ничего.

### Обязательные методы (must implement)

| Метод                                      | Что возвращает                                 | Самый частый пример реализации 2026 |
|--------------------------------------------|------------------------------------------------|-------------------------------------|
| `tableView(_:numberOfRowsInSection:)`      | Количество строк в конкретной секции           | `return items.count` или `data[section].count` |
| `tableView(_:cellForRowAt:)`               | Готовую ячейку для показа по indexPath         | `dequeueReusableCell` → `configure(with:)` → `return cell` |

### Часто реализуемые необязательные методы

| Метод                                      | Что делает                                     | Когда нужен |
|--------------------------------------------|------------------------------------------------|-------------|
| `numberOfSections(in:)`                    | Сколько секций всего                           | Почти всегда (по умолчанию 1) |
| `tableView(_:titleForHeaderInSection:)`    | Заголовок секции (строка сверху)               | Группировка по буквам/датам |
| `tableView(_:titleForFooterInSection:)`    | Подвал секции (строка снизу)                   | Подсказки, итоги |
| `tableView(_:canEditRowAt:)`               | Можно ли редактировать строку                  | Swipe to delete / move |
| `tableView(_:commit:forRowAt:)`            | Обработка удаления/перемещения                 | Редактирование списка |

### Самый современный паттерн 2026 года — с **UITableViewDiffableDataSource**

```swift
final class SettingsViewController: UIViewController {
    
    enum Section: Hashable {
        case account
        case preferences
        case about
    }
    
    enum Item: Hashable {
        case profile(name: String)
        case toggle(title: String, isOn: Bool)
        case link(title: String, url: URL)
    }
    
    typealias DataSource = UITableViewDiffableDataSource<Section, Item>
    typealias Snapshot = NSDiffableDataSourceSnapshot<Section, Item>
    
    private var dataSource: DataSource!
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableView.register(SwitchCell.self, forCellReuseIdentifier: "switch")
        
        view.addSubview(tableView)
        tableView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        configureDataSource()
        applyInitialSnapshot()
    }
    
    private func configureDataSource() {
        dataSource = UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, item in
            switch item {
            case .profile(let name):
                let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
                cell.textLabel?.text = name
                cell.accessoryType = .disclosureIndicator
                return cell
                
            case .toggle(let title, let isOn):
                let cell = tableView.dequeueReusableCell(withIdentifier: "switch", for: indexPath) as! SwitchCell
                cell.configure(title: title, isOn: isOn)
                return cell
                
            case .link(let title, _):
                let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
                cell.textLabel?.text = title
                cell.accessoryType = .disclosureIndicator
                return cell
            }
        }
    }
    
    private func applyInitialSnapshot() {
        var snapshot = Snapshot()
        snapshot.appendSections([.account, .preferences, .about])
        
        snapshot.appendItems([.profile(name: "Alex")], toSection: .account)
        snapshot.appendItems([
            .toggle(title: "Dark Mode", isOn: false),
            .toggle(title: "Notifications", isOn: true)
        ], toSection: .preferences)
        snapshot.appendItems([.link(title: "Privacy Policy", url: URL(string: "https://example.com")!)], toSection: .about)
        
        dataSource.apply(snapshot, animatingDifferences: false)
    }
    
    // Обновление при изменении данных
    func updateDarkMode(isOn: Bool) {
        var snapshot = dataSource.snapshot()
        if let index = snapshot.itemIdentifiers(inSection: .preferences).firstIndex(where: {
            if case .toggle("Dark Mode", _) = $0 { return true }
            return false
        }) {
            snapshot.reloadItems([snapshot.itemIdentifiers[inIndex]])
        }
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

### Короткий чек-лист «Что нужно сделать, чтобы UITableViewDataSource заработал»

1. Создать UITableView (style: .plain / .grouped / .insetGrouped)  
2. Зарегистрировать ячейки  
3. Назначить dataSource (self или отдельный объект)  
4. Реализовать минимум `numberOfRowsInSection` и `cellForRowAt`  
5. Вызвать `reloadData()` или применить snapshot при изменении данных  

### Короткий девиз 2026

> UITableViewDataSource — это когда ты говоришь таблице:  
> «сколько у меня строк и какую ячейку показать на этой строке».  
> В 2026 году **рекомендуется** использовать **DiffableDataSource** — он даёт анимации, производительность и меньше багов.

Удачи с быстрым и красивым списком в Swift! 📜