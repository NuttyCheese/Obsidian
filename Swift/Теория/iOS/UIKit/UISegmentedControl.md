**UISegmentedControl** — это классический элемент управления в [[UIKit]], который представляет собой набор **взаимоисключающих кнопок** (сегментов), из которых пользователь может выбрать **ровно один**.

Это один из самых старых и до сих пор активно используемых контроллеров в iOS-приложениях для быстрого переключения между несколькими (обычно 2–5) вариантами.

### Основные свойства UISegmentedControl (актуально на 2026 год)

| Свойство / Метод                     | Тип / Значение по умолчанию                        | Что делает / зачем нужно                                    | Самый частый сценарий                              |
| ------------------------------------ | -------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| `selectedSegmentIndex`               | [[Int]] (-1 = ничего не выбрано)                   | Индекс текущего выбранного сегмента                         | Основное свойство, к которому привязываются данные |
| `numberOfSegments`                   | `Int` (только для чтения)                          | Количество сегментов                                        | Проверка перед добавлением действий                |
| `segments` (iOS 13+)                 | `[UISegmentedControl.Segment]` (только для чтения) | Доступ к отдельным сегментам (title, image, enabled и т.д.) | Кастомизация отдельных сегментов                   |
| `setTitle(_:forSegmentAt:)`          | `Void`                                             | Установить текст для сегмента                               | Динамическое изменение текста                      |
| `setImage(_:forSegmentAt:)`          | `Void`                                             | Установить иконку (UIImage) для сегмента                    | Иконки вместо текста (iOS 13+)                     |
| `setEnabled(_:forSegmentAt:)`        | `Void`                                             | Включить/выключить отдельный сегмент                        | Отключение недоступных опций                       |
| `apportionsSegmentWidthsByContent`   | [[Bool]] (false)                                   | Автоматически подгонять ширину сегментов под контент        | Когда текст разной длины                           |
| `isMomentary`                        | `Bool` (false)                                     | Сегмент не остаётся выделенным после нажатия                | Редко, для кнопок-действий                         |
| `backgroundColor` / `tintColor`      | [[UIColor]]`?`                                     | Цвет фона и акцента (влияет на выделенный сегмент)          | Кастомизация под бренд                             |
| `selectedSegmentTintColor` (iOS 13+) | `UIColor?`                                         | Цвет выделенного сегмента                                   | Современный стиль                                  |

### Самый популярный и рекомендуемый паттерн 2026 года (UIKit + MVVM + Combine)

```swift
import UIKit
import Combine

class FilterViewController: UIViewController {
    
    private let viewModel = FilterViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var segmentedControl: UISegmentedControl = {
        let items = ["Все", "Активные", "Завершённые"]
        let sc = UISegmentedControl(items: items)
        sc.selectedSegmentIndex = viewModel.selectedFilter.rawValue
        sc.selectedSegmentTintColor = .systemBlue
        sc.backgroundColor = .systemGray6
        sc.layer.cornerRadius = 8
        sc.clipsToBounds = true
        sc.addTarget(self, action: #selector(segmentChanged), for: .valueChanged)
        sc.translatesAutoresizingMaskIntoConstraints = false
        return sc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        view.addSubview(segmentedControl)
        
        NSLayoutConstraint.activate([
            segmentedControl.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
            segmentedControl.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            segmentedControl.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            segmentedControl.heightAnchor.constraint(equalToConstant: 44)
        ])
    }
    
    private func bindViewModel() {
        // Двусторонняя связь: segmentedControl ↔ ViewModel
        viewModel.$selectedFilter
            .map { $0.rawValue }
            .assign(to: \.selectedSegmentIndex, on: segmentedControl)
            .store(in: &cancellables)
        
        // Реакция на изменение в ViewModel (например, обновление списка)
        viewModel.$selectedFilter
            .sink { [weak self] filter in
                self?.updateList(for: filter)
            }
            .store(in: &cancellables)
    }
    
    @objc private func segmentChanged(_ sender: UISegmentedControl) {
        if let filter = FilterType(rawValue: sender.selectedSegmentIndex) {
            viewModel.selectedFilter = filter
        }
    }
    
    private func updateList(for filter: FilterType) {
        // здесь обновляем таблицу / коллекцию
        print("Фильтр изменён на:", filter.title)
    }
}

// ViewModel
enum FilterType: Int, CaseIterable {
    case all = 0
    case active
    case completed
    
    var title: String {
        switch self {
        case .all: return "Все"
        case .active: return "Активные"
        case .completed: return "Завершённые"
        }
    }
}

@MainActor
class FilterViewModel: ObservableObject {
    @Published var selectedFilter: FilterType = .all
}
```

### Современные альтернативы UISegmentedControl в 2026 году

| Компонент / Способ                            | Когда лучше UISegmentedControl                                 | Минимальная версия | Плюсы                                       |
| --------------------------------------------- | -------------------------------------------------------------- | ------------------ | ------------------------------------------- |
| **Segmented** в [[SwiftUI]]                   | Новый проект или [[SwiftUI]]-экран                             | iOS 13+            | Декларативный стиль, проще привязка         |
| **UISegmentedControl + [[Combine]]**          | [[UIKit]] + [[MVVM (Model-View-ViewModel) Architecture\|MVVM]] | iOS 13+            | Реактивность, двусторонняя привязка         |
| **[[UITabBar]]** / **[[UITabBarController]]** | Постоянное переключение между основными экранами               | iOS 13+            | Стандартный таб-бар                         |
| **[[UIPageControl]] + [[UIScrollView]]**      | Когда нужно много вариантов + свайп                            | iOS 13+            | Горизонтальный свайп                        |
| **Menu / [[UIMenu]]** (iOS 14+)               | Когда вариантов много или нужны подменю                        | iOS 14+            | Более современный вид, контекстные действия |

### Лучшие практики UISegmentedControl в 2026 году

- **Всегда** задавайте `selectedSegmentIndex` при инициализации — иначе ничего не выбрано  
- **Используйте** `.selectedSegmentTintColor` и `.backgroundColor` для кастомизации (iOS 13+)  
- **Для иконок** — используйте `setImage(_:forSegmentAt:)` — текст и иконки можно комбинировать  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue` для каждого сегмента  
- **Для Combine** — привязывайте `selectedSegmentIndex` через `.publisher(for: \.selectedSegmentIndex)` или `@Published` в ViewModel  
- **Для SwiftUI** — используйте нативный `Picker` со стилем `.segmented` — UISegmentedControl нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UISegmentedControl для выбора фильтра задач (Все / Активные / Завершённые)
private lazy var filterSegmentedControl: UISegmentedControl = {
    let sc = UISegmentedControl(items: ["Все", "Активные", "Завершённые"])
    sc.selectedSegmentIndex = viewModel.selectedFilter.rawValue
    return sc
}()
```

**Короткий итог 2026**:
> `UISegmentedControl` — классический контроллер для **быстрого выбора одного варианта** из небольшого фиксированного набора.  
> В 2026 году:  
> - ключевые свойства — `selectedSegmentIndex`, `minimumValue`, `maximumValue`, `stepValue`  
> - идеален для фильтров, вкладок, выбора единиц измерения, статуса  
> - в SwiftUI — используйте `Picker(.segmented)`  
> - в UIKit — синхронизируйте с `@Published` через Combine или target-action  
> - это **надёжный** и **доступный** элемент интерфейса, который до сих пор активно используется  
