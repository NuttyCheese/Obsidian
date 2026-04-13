**`UISearchResultsUpdating`** — это **протокол** в [[UIKit]], который отвечает за **обновление результатов поиска** при каждом изменении текста в строке поиска ([[UISearchBar]]) внутри [[UISearchController]].

Это **самый важный** и **самый часто реализуемый** протокол при работе с `UISearchController`.  
Без него поиск просто не будет фильтровать данные — строка поиска будет пустой декорацией.

### Когда и где используется UISearchResultsUpdating

| Ситуация                                          | Кто реализует протокол                                          | Самый частый сценарий 2025–2026       |
| ------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------- |
| Поиск в той же таблице/коллекции                  | Сам контроллер с таблицей ([[self]])                            | Фильтрация текущего списка            |
| Поиск с отдельным экраном результатов             | Отдельный контроллер результатов                                | Полноэкранный поиск (как в App Store) |
| Поиск в [[SwiftUI]] через [[UIViewRepresentable]] | Coordinator или внешний объект                                  | Смешанные UIKit + [[SwiftUI]] проекты |
| Поиск в Navigation Bar                            | Обычно [[UIViewController]] с `navigationItem.searchController` | Стандартный поиск в навигации         |

### Единственный обязательный метод протокола

```swift
protocol UISearchResultsUpdating : NSObjectProtocol {
    func updateSearchResults(for searchController: UISearchController)
}
```

Этот метод вызывается **каждый раз**, когда пользователь:
- печатает/стирает символ в строке поиска
- нажимает на крестик очистки
- меняет scope button (если они есть)

**Важно**: метод вызывается **очень часто** (иногда несколько раз в секунду при быстром наборе).  
Поэтому внутри него нужно делать **эффективную фильтрацию** и **минимизировать UI-обновления**.

### Самый популярный и рекомендуемый паттерн 2026 года

#### Вариант 1. Поиск в той же таблице (самый частый)

```swift
class FruitsListViewController: UIViewController, UISearchResultsUpdating {
    
    private let tableView = UITableView()
    private var searchController: UISearchController!
    
    private let allFruits = ["Apple", "Banana", "Cherry", "Date", "Elderberry", "Fig", "Grape"]
    private var filteredFruits: [String] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Фрукты"
        filteredFruits = allFruits
        
        setupTableView()
        setupSearchController()
    }
    
    private func setupTableView() {
        tableView.dataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        view = tableView
    }
    
    private func setupSearchController() {
        searchController = UISearchController(searchResultsController: nil)
        
        searchController.searchResultsUpdater = self                    // ← здесь подключаем
        searchController.obscuresBackgroundDuringPresentation = false
        searchController.hidesNavigationBarDuringPresentation = false
        searchController.searchBar.placeholder = "Поиск фрукта..."
        
        navigationItem.searchController = searchController
        navigationItem.hidesSearchBarWhenScrolling = false
    }
    
    // Самый важный метод протокола
    func updateSearchResults(for searchController: UISearchController) {
        // Получаем текст поиска (нижний регистр для case-insensitive)
        guard let searchText = searchController.searchBar.text?.lowercased(),
              !searchText.isEmpty else {
            filteredFruits = allFruits
            tableView.reloadData()
            return
        }
        
        // Эффективная фильтрация
        filteredFruits = allFruits.filter { $0.lowercased().contains(searchText) }
        
        // Обновляем таблицу
        tableView.reloadData()
    }
}

// MARK: - UITableViewDataSource
extension FruitsListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        filteredFruits.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = filteredFruits[indexPath.row]
        return cell
    }
}
```

### Оптимизация updateSearchResults (очень важно в 2026)

Метод вызывается **очень часто** → вот как сделать его производительным:

```swift
private var searchWorkItem: DispatchWorkItem?

func updateSearchResults(for searchController: UISearchController) {
    // Отменяем предыдущую задачу, если пользователь быстро печатает
    searchWorkItem?.cancel()
    
    let workItem = DispatchWorkItem { [weak self] in
        guard let self else { return }
        
        guard let text = searchController.searchBar.text?.lowercased(),
              !text.isEmpty else {
            self.filteredFruits = self.allFruits
            DispatchQueue.main.async { self.tableView.reloadData() }
            return
        }
        
        // Тяжёлая фильтрация (например, по нескольким полям)
        self.filteredFruits = self.allFruits.filter {
            $0.lowercased().contains(text) || $0.localizedCaseInsensitiveContains(text)
        }
        
        DispatchQueue.main.async { self.tableView.reloadData() }
    }
    
    // Задержка 0.3 сек — пользователь успевает допечатать
    searchWorkItem = workItem
    DispatchQueue.global(qos: .userInitiated).asyncAfter(deadline: .now() + 0.3, execute: workItem)
}
```

### Лучшие практики UISearchResultsUpdating в 2026 году

- **Всегда** реализуйте `updateSearchResults(for:)` — без него поиск не работает  
- **Делайте** фильтрацию **асинхронной** + с debounce (задержкой) — особенно если список большой  
- **Используйте** [[DispatchQueue]].[[main]].[[async]] для обновления UI — метод может вызываться не на главном потоке  
- **Для производительности** — применяйте [[UITableViewDiffableDataSource]] или `UICollectionViewDiffableDataSource` — они обновляют таблицу умно  
- **Для SwiftUI** — используйте `.searchable(text:)` — `UISearchResultsUpdating` не нужен  
- **Для scope buttons** — обрабатывайте их в `searchBar(_:selectedScopeButtonIndexDidChange:)`  
- **Документируйте** — пишите комментарий «UISearchResultsUpdating — фильтрация списка в реальном времени при изменении поискового запроса»

**Короткий итог 2026**:
> `UISearchResultsUpdating` — это **единственный протокол**, через который `UISearchController` сообщает: «Текст поиска изменился — обнови результаты».  
> В 2026 году:  
> - единственный метод — `updateSearchResults(for:)`  
> - вызывается **очень часто** — делай фильтрацию лёгкой и с debounce  
> - подключается через `searchController.searchResultsUpdater = self`  
> - это **обязательный** компонент любого поиска в UIKit-приложении  
