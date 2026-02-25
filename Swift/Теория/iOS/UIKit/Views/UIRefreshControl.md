**UIRefreshControl** — это системный компонент [[UIKit]] для реализации **pull-to-refresh** (потянуть вниз для обновления) в прокручиваемых представлениях.

С iOS 10 (2016) он стал одним из самых популярных и надёжных способов добавить «тяни для обновления» в [[UIScrollView]], [[UITableView]], [[UICollectionView]].

В 2026 году UIRefreshControl **по-прежнему актуален** и остаётся **стандартным** решением в UIKit-приложениях.

### Основные свойства UIRefreshControl (самые важные в 2026)

| Свойство                          | Тип / Значение по умолчанию       | Что делает / зачем нужно                             | Самый частый сценарий    |
| --------------------------------- | --------------------------------- | ---------------------------------------------------- | ------------------------ |
| `isRefreshing`                    | [[Bool]] (false)                  | Показывает, выполняется ли обновление сейчас         | Проверка состояния       |
| `attributedTitle`                 | [[NSAttributedString]]`?`         | Текст над спиннером (например «Обновление...»)       | Локализация и стилизация |
| `tintColor`                       | [[UIColor]]`?` (системный акцент) | Цвет спиннера и текста                               | Кастомизация под бренд   |
| `backgroundColor`                 | `UIColor?`                        | Цвет фона спиннера (редко используют)                | —                        |
| `refreshControl` (в UIScrollView) | `UIRefreshControl?`               | Ссылка на refresh control, прикреплённый к скроллвью | Добавление/удаление      |
| `beginRefreshing()`               | `Void`                            | Программно начать анимацию обновления                | Обновление по кнопке     |
| `endRefreshing()`                 | `Void`                            | Завершить анимацию (обязательно!)                    | После загрузки данных    |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UIScrollView]] / [[UITableView]] / [[UICollectionView]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

class FeedViewController: UIViewController {
    
    private let viewModel = FeedViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var tableView: UITableView = {
        let tv = UITableView(frame: .zero, style: .plain)
        tv.translatesAutoresizingMaskIntoConstraints = false
        tv.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        return tv
    }()
    
    private lazy var refreshControl: UIRefreshControl = {
        let rc = UIRefreshControl()
        rc.attributedTitle = NSAttributedString(string: "Обновление ленты...")
        rc.tintColor = .systemBlue
        rc.addTarget(self, action: #selector(refreshData), for: .valueChanged)
        return rc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Лента"
        
        tableView.refreshControl = refreshControl
        view.addSubview(tableView)
        
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])
        
        setupTableView()
        bindViewModel()
        
        // Автоматический refresh при первом появлении
        refreshData()
    }
    
    private func setupTableView() {
        tableView.dataSource = self
        tableView.delegate = self
    }
    
    private func bindViewModel() {
        viewModel.$items
            .receive(on: DispatchQueue.main)
            .sink { [weak self] items in
                self?.tableView.reloadData()
                self?.refreshControl.endRefreshing()
            }
            .store(in: &cancellables)
        
        viewModel.$error
            .receive(on: DispatchQueue.main)
            .sink { [weak self] error in
                if let error {
                    self?.showError(error)
                    self?.refreshControl.endRefreshing()
                }
            }
            .store(in: &cancellables)
    }
    
    @objc private func refreshData() {
        viewModel.refresh()
    }
}

// ViewModel (пример)
@MainActor
class FeedViewModel: ObservableObject {
    @Published var items: [String] = []
    @Published var error: Error?
    
    func refresh() {
        error = nil
        
        Task {
            do {
                // имитация сети
                try await Task.sleep(nanoseconds: 1_500_000_000)
                items = (1...20).map { "Элемент \($0)" }
            } catch {
                self.error = error
            }
        }
    }
}

extension FeedViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        viewModel.items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = viewModel.items[indexPath.row]
        return cell
    }
}
```

### Современные альтернативы UIRefreshControl в 2026 году

| Компонент / Способ                              | Когда лучше UIRefreshControl            | Минимальная версия | Плюсы                                                   |
| ----------------------------------------------- | --------------------------------------- | ------------------ | ------------------------------------------------------- |
| **Refreshable** в SwiftUI                       | Новый проект или SwiftUI-экран          | iOS 15+            | Декларативный стиль, `.refreshable { await refresh() }` |
| **UIRefreshControl + Combine**                  | UIKit + MVVM                            | iOS 10+            | Реактивность, удобная обработка ошибок                  |
| **Custom pull-to-refresh** ([[SPM]] библиотеки) | Очень специфический дизайн или анимация | iOS 13+            | Полная кастомизация (Lottie, кастомные спиннеры)        |
| **Refreshable + ScrollView** в SwiftUI          | Полноценная лента с кастомным стилем    | iOS 15+            | Лучшая интеграция с NavigationStack                     |

### Лучшие практики UIRefreshControl в 2026 году

- **Всегда** вызывайте `endRefreshing()` после завершения загрузки (даже при ошибке)  
- **Задавайте** `attributedTitle` для локализации и стилизации («Обновление...»)  
- **Для цвета** — используйте `tintColor` (спиннер) и системные цвета для фона  
- **Для Combine / async** — запускайте обновление в `refreshData()` и завершайте в `.sink` / `await`  
- **Для доступности** — `accessibilityLabel = "Обновить ленту"`  
- **Для SwiftUI** — используйте `.refreshable { await viewModel.refresh() }` — UIRefreshControl нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UIRefreshControl для pull-to-refresh ленты новостей
private lazy var refreshControl: UIRefreshControl = {
    let rc = UIRefreshControl()
    rc.attributedTitle = NSAttributedString(string: "Обновление...")
    rc.tintColor = .systemBlue
    rc.addTarget(self, action: #selector(refreshData), for: .valueChanged)
    return rc
}()
```

**Короткий итог 2026**:
> `UIRefreshControl` — системный контроллер для **pull-to-refresh** в прокручиваемых представлениях.  
> В 2026 году:  
> - ключевые методы — `beginRefreshing()`, `endRefreshing()`  
> - добавляется через `scrollView.refreshControl = rc`  
> - идеален для лент, списков, таблиц, коллекций  
> - в SwiftUI — заменяется на `.refreshable { ... }`  
> - это **надёжный** и **стандартный** способ добавить обновление по свайпу вниз  
