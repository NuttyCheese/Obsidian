**`UISearchController`** — это контроллер в [[UIKit]] (доступен с iOS 8, 2014), который управляет **полноценным поисковым интерфейсом** в приложении.

Он берёт на себя:
- строку поиска (search bar)
- результаты поиска
- переход между состоянием «есть поиск» и «нет поиска»
- интеграцию с навигацией (особенно в [[UINavigationController]])

В 2025–2026 годах это **самый рекомендуемый** и **самый часто используемый** способ реализовать поиск в таблицах, коллекциях и других списках.

### Основные компоненты UISearchController

| Компонент                              | Тип / Класс                           | Что делает / зачем нужен                                                   | Самый частый сценарий |
| -------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------- | --------------------- |
| `searchBar`                            | [[UISearchBar]]                       | Сама строка поиска (текст, плейсхолдер, scope buttons и т.д.)              | Основной элемент      |
| `searchResultsController`              | [[UIViewController]]`?                | Контроллер, который показывает результаты поиска (часто отдельная таблица) | Результаты поиска     |
| `searchResultsUpdater`                 | [[UISearchResultsUpdating]] (делегат) | Вызывается при каждом изменении текста в строке поиска                     | Фильтрация данных     |
| `delegate`                             | [[UISearchControllerDelegate]]        | События показа/скрытия поискового интерфейса                               | Анимации, логика      |
| `obscuresBackgroundDuringPresentation` | [[Bool]]                              | Затемняет фон при показе результатов (по умолчанию true)                   | UX как в App Store    |
| `hidesNavigationBarDuringPresentation` | `Bool`                                | Скрывает navigation bar при активном поиске                                | Современный стиль     |

### Самый популярный паттерн 2026 года ([[UITableView]] / [[UICollectionView]] + UISearchController)

```swift
import UIKit

class SearchableListViewController: UIViewController {
    
    private let tableView = UITableView()
    private var searchController: UISearchController!
    
    // Данные
    private var allItems = ["Apple", "Banana", "Cherry", "Date", "Elderberry", "Fig", "Grape"]
    private var filteredItems: [String] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Фрукты"
        filteredItems = allItems
        
        setupTableView()
        setupSearchController()
    }
    
    private func setupTableView() {
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        view = tableView
    }
    
    private func setupSearchController() {
        // 1. Создаём контроллер поиска
        searchController = UISearchController(searchResultsController: nil)
        
        // 2. Настраиваем поведение
        searchController.searchResultsUpdater = self
        searchController.obscuresBackgroundDuringPresentation = false
        searchController.hidesNavigationBarDuringPresentation = false
        searchController.searchBar.placeholder = "Поиск фрукта..."
        
        // 3. Интегрируем в navigation bar (самый популярный способ)
        navigationItem.searchController = searchController
        navigationItem.hidesSearchBarWhenScrolling = false  // всегда видна
        
        // Опционально: scope buttons (фильтры)
        searchController.searchBar.scopeButtonTitles = ["Все", "Начинается с A", "Длинные"]
        searchController.searchBar.delegate = self
    }
}

// MARK: - UISearchResultsUpdating
extension SearchableListViewController: UISearchResultsUpdating {
    
    func updateSearchResults(for searchController: UISearchController) {
        guard let text = searchController.searchBar.text?.lowercased(), !text.isEmpty else {
            filteredItems = allItems
            tableView.reloadData()
            return
        }
        
        filteredItems = allItems.filter { $0.lowercased().contains(text) }
        tableView.reloadData()
    }
}

// MARK: - UISearchBarDelegate (для scope buttons)
extension SearchableListViewController: UISearchBarDelegate {
    
    func searchBar(_ searchBar: UISearchBar, selectedScopeButtonIndexDidChange selectedScope: Int) {
        // Пример: фильтрация по scope
        switch selectedScope {
        case 1: // Начинается с A
            filteredItems = allItems.filter { $0.lowercased().hasPrefix("a") }
        case 2: // Длинные
            filteredItems = allItems.filter { $0.count > 5 }
        default:
            filteredItems = allItems
        }
        
        tableView.reloadData()
    }
}

// MARK: - UITableViewDataSource
extension SearchableListViewController: UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        filteredItems.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = filteredItems[indexPath.row]
        return cell
    }
}
```

### Современные тренды и лучшие практики 2026 года

1. **Интеграция с navigation bar** (самый популярный способ)

```swift
navigationItem.searchController = searchController
navigationItem.hidesSearchBarWhenScrolling = false  // поиск всегда виден
```

2. **Результаты в том же контроллере** (searchResultsController = [[nil]])

```swift
searchController.searchResultsController = nil
// тогда updateSearchResults фильтрует текущую таблицу
```

3. **Результаты в отдельном контроллере** (классический подход)

```swift
let resultsVC = SearchResultsViewController()
searchController = UISearchController(searchResultsController: resultsVC)
searchController.searchResultsUpdater = resultsVC
```

4. **Scope buttons + search bar** (фильтры + поиск)

```swift
searchController.searchBar.scopeButtonTitles = ["Все", "Фрукты", "Овощи"]
searchController.searchBar.delegate = self
```

5. **Кастомный вид** (appearance)

```swift
let appearance = UISearchBarAppearance()
appearance.backgroundColor = .systemGray6
searchController.searchBar.searchBarStyle = .minimal
searchController.searchBar.searchTextField.backgroundColor = .systemGray6
```

### Лучшие практики UISearchController в 2026 году

- **Всегда** используйте `navigationItem.searchController` — это современный и рекомендуемый способ  
- **Задавайте** `hidesSearchBarWhenScrolling = false` — поиск всегда виден (стандарт Apple 2024–2026)  
- **Для простых списков** — `searchResultsController = nil` + фильтрация в `updateSearchResults`  
- **Для сложных результатов** — отдельный `searchResultsController`  
- **Для SwiftUI** — используйте `searchable(text:)` — UISearchController нужен только в смешанных проектах  
- **Для производительности** — обновляйте таблицу эффективно (DiffableDataSource или reloadSections)  
- **Для доступности** — задавайте `searchBar.placeholder` и `accessibilityLabel`  
- **Документируйте** — пишите комментарий «UISearchController — поиск по списку фруктов с фильтрацией в реальном времени»

**Короткий итог 2026**:
> `UISearchController` — это **системный контроллер поиска** для списков и коллекций.  
> В 2026 году:  
> - привязывается через `navigationItem.searchController`  
> - обновление результатов — через `searchResultsUpdater`  
> - поддерживает scope buttons, отдельный results controller, кастомный вид  
> - это **единственный рекомендуемый** способ реализовать поиск в UIKit-приложениях  
