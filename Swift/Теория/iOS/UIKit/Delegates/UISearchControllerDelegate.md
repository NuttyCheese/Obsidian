**UISearchControllerDelegate** — это протокол в [[UIKit]], который определяет методы-делегаты для обработки **событий жизненного цикла** поискового интерфейса, управляемого [[UISearchController]].

Протокол **не обязателен** — большинство приложений вообще его не реализуют, потому что основной функционал поиска обеспечивается через `searchResultsUpdater` и `searchBar.delegate`.  
Но `UISearchControllerDelegate` нужен, когда требуется тонкая настройка поведения при **появлении/исчезновении** поисковой панели, особенно в сценариях с анимациями, кастомным затемнением фона или сложной навигацией.

### Методы протокола (полный список, актуально на 2026 год)

| Метод делегата                    | Когда вызывается                                                         | Что обычно делают внутри метода                                                     | Обязателен? | Частота использования       |
| --------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- | ----------- | --------------------------- |
| `searchControllerWillPresent(_:)` | Непосредственно **перед** началом презентации поискового интерфейса      | Подготовка UI (скрытие элементов, начало анимаций, логи)                            | Нет         | Средняя                     |
| `searchControllerDidPresent(_:)`  | **После** завершения презентации (анимация закончилась)                  | Действия, которые должны произойти после появления (фокус на search bar, аналитика) | Нет         | Низкая                      |
| `searchControllerWillDismiss(_:)` | **Перед** началом скрытия поискового интерфейса                          | Сохранение состояния поиска, отмена фокуса, подготовка к уходу                      | Нет         | Средняя                     |
| `searchControllerDidDismiss(_:)`  | **После** завершения скрытия (анимация закончилась)                      | Очистка временных данных, сброс фильтров, аналитика                                 | Нет         | Низкая                      |
| `willPresentSearchController(_:)` | **Устаревший** (до [[iOS]] 13), эквивалент `searchControllerWillPresent` | —                                                                                   | Нет         | Никогда (используйте новый) |
| `didPresentSearchController(_:)`  | **Устаревший** (до iOS 13)                                               | —                                                                                   | Нет         | Никогда                     |
| `willDismissSearchController(_:)` | **Устаревший** (до iOS 13)                                               | —                                                                                   | Нет         | Никогда                     |
| `didDismissSearchController(_:)`  | **Устаревший** (до iOS 13)                                               | —                                                                                   | Нет         | Никогда                     |

**Важно**:  
С iOS 13 методы переименованы и получили префикс `searchController` → используйте **новые** имена.  
Старые методы помечены как deprecated и будут удалены в будущем.

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
class SearchViewController: UIViewController, UISearchControllerDelegate, UISearchResultsUpdating, UISearchBarDelegate {
    
    private var searchController: UISearchController!
    private let tableView = UITableView()
    
    private var allItems = ["Apple", "Banana", "Cherry", "Date", "Elderberry"]
    private var filteredItems: [String] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Поиск"
        filteredItems = allItems
        
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
        searchController.searchResultsUpdater = self
        searchController.delegate = self                    // ← здесь подключаем делегат
        searchController.obscuresBackgroundDuringPresentation = false
        searchController.hidesNavigationBarDuringPresentation = false
        searchController.searchBar.placeholder = "Поиск фрукта..."
        
        navigationItem.searchController = searchController
        navigationItem.hidesSearchBarWhenScrolling = false
    }
    
    // MARK: - UISearchResultsUpdating
    func updateSearchResults(for searchController: UISearchController) {
        guard let text = searchController.searchBar.text?.lowercased(), !text.isEmpty else {
            filteredItems = allItems
            tableView.reloadData()
            return
        }
        
        filteredItems = allItems.filter { $0.lowercased().contains(text) }
        tableView.reloadData()
    }
    
    // MARK: - UISearchControllerDelegate
    func searchControllerWillPresent(_ searchController: UISearchController) {
        print("Поисковая панель будет показана")
        // Можно подготовить UI, скрыть лишние элементы
    }
    
    func searchControllerDidPresent(_ searchController: UISearchController) {
        print("Поисковая панель полностью показана")
        // Можно установить фокус на search bar
        searchController.searchBar.becomeFirstResponder()
    }
    
    func searchControllerWillDismiss(_ searchController: UISearchController) {
        print("Поисковая панель будет скрыта")
        // Сохраняем состояние поиска, если нужно
    }
    
    func searchControllerDidDismiss(_ searchController: UISearchController) {
        print("Поисковая панель полностью скрыта")
        // Очищаем временные фильтры, если нужно
        filteredItems = allItems
        tableView.reloadData()
    }
}
```

### Лучшие практики UISearchControllerDelegate в 2026 году

- **Реализуйте** делегат **только** если вам нужно реагировать на появление/исчезновение поисковой панели  
- **Чаще всего** достаточно `searchResultsUpdater` и `searchBar.delegate` — делегат UISearchControllerDelegate нужен в ~10–15% случаев  
- **Используйте** `willPresent` / `didPresent` для:
  - установки фокуса на `searchBar.becomeFirstResponder()`
  - скрытия/показа дополнительных элементов интерфейса
  - логирования / аналитики
- **Используйте** `willDismiss` / `didDismiss` для:
  - сброса фильтров
  - сохранения последнего поискового запроса
  - очистки временных данных
- **Для [[SwiftUI]]** — используйте модификатор `.searchable(text:)` — UISearchControllerDelegate не нужен  
- **Для производительности** — избегайте тяжёлых операций в `willPresent`/`willDismiss` — они вызываются синхронно во время анимации  
- **Документируйте** — пишите комментарий «UISearchControllerDelegate — обработка событий показа/скрытия поисковой панели (фокус, сброс фильтров)»

**Короткий итог 2026**:
> `UISearchControllerDelegate` — протокол для обработки **событий жизненного цикла** поисковой панели.  
> В 2026 году:  
> - основные методы — `searchControllerWillPresent`, `searchControllerDidPresent`, `searchControllerWillDismiss`, `searchControllerDidDismiss`  
> - чаще всего достаточно `searchResultsUpdater` — делегат нужен только для тонкой настройки появления/исчезновения  
> - используется в связке с `navigationItem.searchController`  
> - это **вспомогательный**, но **очень полезный** инструмент для кастомизации UX поиска  
