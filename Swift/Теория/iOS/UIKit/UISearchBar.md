**UISearchBar** — это встроенный элемент интерфейса UIKit, предназначенный для реализации **поиска** в приложении. Это классический контроллер поиска, который используется в iOS уже более 15 лет и до сих пор остаётся одним из самых популярных способов добавить строку поиска в UIKit-приложениях.

В 2026 году UISearchBar **по-прежнему актуален**, особенно в чистом UIKit или смешанных проектах, хотя в новых приложениях на SwiftUI чаще используют модификатор `.searchable()`.

### Основные свойства UISearchBar (самые важные в 2026)

| Свойство                          | Тип / Значение по умолчанию                          | Что делает / зачем нужно                                      | Самый частый сценарий |
|-----------------------------------|-------------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `text`                            | `String?`                                             | Текущий текст в строке поиска                                 | Основное свойство для ввода/чтения запроса |
| `placeholder`                     | `String?`                                             | Подсказка, когда поле пустое                                  | "Поиск", "Введите запрос" и т.д. |
| `prompt`                          | `String?`                                             | Дополнительный текст над строкой поиска                       | Редко, для пояснений |
| `scopeButtonTitles`               | `[String]?`                                           | Заголовки кнопок фильтров (scope)                             | "Все", "Фильм", "Сериал" |
| `selectedScopeButtonIndex`        | `Int` (0)                                             | Индекс выбранного scope                                       | Фильтрация по категориям |
| `showsScopeBar`                   | `Bool` (false)                                        | Показывать ли панель scope-кнопок                             | Включается при наличии scopeButtonTitles |
| `showsCancelButton`               | `Bool` (false)                                        | Показывать кнопку "Отмена"                                    | Обычно включают при фокусе |
| `searchBarStyle`                  | `UISearchBar.Style` (.default / .prominent / .minimal) | Визуальный стиль (с iOS 13+)                                  | `.minimal` — современный плоский вид |
| `barTintColor` / `tintColor`      | `UIColor?`                                            | Цвет бара и акцента                                           | Кастомизация под бренд |
| `searchTextField` (iOS 13+)       | `UITextField` (только чтение)                         | Прямой доступ к полю ввода                                    | Тонкая кастомизация (backgroundColor, cornerRadius) |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UISearchBar + UISearchController + Combine + MVVM)

```swift
import UIKit
import Combine

class SearchViewController: UIViewController {
    
    private let viewModel = SearchViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var searchController: UISearchController = {
        let sc = UISearchController(searchResultsController: nil)
        sc.searchResultsUpdater = self
        sc.obscuresBackgroundDuringPresentation = false
        sc.hidesNavigationBarDuringPresentation = false
        sc.searchBar.placeholder = "Поиск фильмов, сериалов..."
        sc.searchBar.scopeButtonTitles = ["Все", "Фильмы", "Сериалы"]
        sc.searchBar.delegate = self
        return sc
    }()
    
    private lazy var tableView: UITableView = {
        let tv = UITableView()
        tv.translatesAutoresizingMaskIntoConstraints = false
        return tv
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Поиск"
        navigationItem.searchController = searchController
        navigationItem.hidesSearchBarWhenScrolling = false
        
        view.addSubview(tableView)
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$searchResults
            .receive(on: DispatchQueue.main)
            .sink { [weak self] results in
                self?.tableView.reloadData()
            }
            .store(in: &cancellables)
    }
}

extension SearchViewController: UISearchResultsUpdating {
    func updateSearchResults(for searchController: UISearchController) {
        guard let text = searchController.searchBar.text, !text.isEmpty else {
            viewModel.searchResults = []
            return
        }
        
        let selectedScope = searchController.searchBar.selectedScopeButtonIndex
        viewModel.performSearch(query: text, scope: selectedScope)
    }
}

extension SearchViewController: UISearchBarDelegate {
    func searchBar(_ searchBar: UISearchBar, selectedScopeButtonIndexDidChange selectedScope: Int) {
        // Реакция на смену scope (фильтра)
        if let text = searchBar.text, !text.isEmpty {
            viewModel.performSearch(query: text, scope: selectedScope)
        }
    }
}

// ViewModel
@MainActor
class SearchViewModel: ObservableObject {
    @Published var searchResults: [String] = []  // или ваша модель
    
    func performSearch(query: String, scope: Int) {
        // Здесь реальный поиск (API, Core Data, локальная фильтрация)
        searchResults = ["Результат 1 для \(query)", "Результат 2"]
    }
}
```

### Современные альтернативы UISearchBar в 2026 году

| Компонент / Способ                        | Когда лучше UISearchBar                            | Минимальная версия | Плюсы |
|-------------------------------------------|-----------------------------------------------------|---------------------|-------|
| **.searchable()** в SwiftUI               | Новый проект или SwiftUI-экран                     | iOS 15+             | Декларативный стиль, интеграция с NavigationStack |
| **UISearchController** (с UISearchBar)    | UIKit + полноценный поиск с результатами            | iOS 13+             | Лучшая интеграция с navigation bar |
| **Custom UITextField + button**           | Очень специфический дизайн поиска                   | iOS 13+             | Полная кастомизация |
| **SF Symbols + UIButton**                 | Минималистичный поиск (иконка лупы)                | iOS 13+             | Лёгкий вес, современный вид |

### Лучшие практики UISearchBar в 2026 году

- **Предпочитайте** `UISearchController` вместо голого `UISearchBar` — это стандартный способ интеграции поиска в navigation bar  
- **Всегда** задавайте `placeholder` — это критично для UX  
- **Для scope-фильтров** — используйте `scopeButtonTitles` и `selectedScopeButtonIndex`  
- **Для отмены** — включайте `showsCancelButton = true` (или вручную через delegate)  
- **Для Combine** — используйте `.publisher(for: \.text)` или `searchController.searchBar.textPublisher` (расширение)  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint`  
- **Для SwiftUI** — используйте `.searchable(text: $searchText)` — UISearchBar нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UISearchController с поисковой строкой и scope-фильтрами (Все / Фильмы / Сериалы)
private lazy var searchController: UISearchController = {
    let sc = UISearchController(searchResultsController: nil)
    sc.searchBar.placeholder = "Поиск фильмов, сериалов..."
    sc.searchBar.scopeButtonTitles = ["Все", "Фильмы", "Сериалы"]
    return sc
}()
```

**Короткий итог 2026**:
> `UISearchBar` — классическая строка поиска в UIKit.  
> В 2026 году:  
> - почти всегда используется через **UISearchController**  
> - ключевые свойства — `text`, `placeholder`, `scopeButtonTitles`, `showsCancelButton`  
> - идеален для фильтров, поиска по списку, интеграции в navigation bar  
> - в SwiftUI — заменяется на `.searchable()`  
> - это **надёжный** и **стандартный** элемент поиска, который до сих пор активно используется  

Удачи с удобным и быстрым поиском в твоём приложении! 🔍