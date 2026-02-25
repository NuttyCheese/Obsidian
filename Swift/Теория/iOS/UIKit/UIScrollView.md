**UIScrollView** — это один из самых фундаментальных и мощных компонентов [[UIKit]], который позволяет отображать контент, **превышающий размер экрана**, с возможностью **прокрутки** (scrolling), масштабирования (zooming), отскока (bouncing) и других интерактивных возможностей.

В 2026 году UIScrollView остаётся **основой** практически всех прокручиваемых интерфейсов в iOS-приложениях: таблицы, коллекции, полноэкранные изображения, ленты новостей, профили, формы и т.д.

### Основные свойства UIScrollView (самые важные в 2026)

| Свойство                                                          | Тип / Значение по умолчанию                                | Что делает / зачем нужно                                         | Самый частый сценарий                             |
| ----------------------------------------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------- |
| `contentSize`                                                     | [[CGSize]] (0,0)                                           | Размер всего контента (ширина × высота)                          | Обязательно задавать, иначе прокрутка не работает |
| `contentOffset`                                                   | [[CGPoint]] (0,0)                                          | Текущая позиция прокрутки (x — горизонтально, y — вертикально)   | Чтение/установка позиции скролла                  |
| `contentInset`                                                    | [[UIEdgeInsets]] (0)                                       | Отступы внутри контента (от краёв скроллвью)                     | Учёт safe area, navigation bar, tab bar           |
| `contentInsetAdjustmentBehavior`                                  | `UIScrollView.ContentInsetAdjustmentBehavior` (.automatic) | Как автоматически учитывать safe area и navigation bar           | `.automatic` — стандарт в iOS 11+                 |
| `showsHorizontalScrollIndicator` / `showsVerticalScrollIndicator` | [[Bool]] (true)                                            | Показывать полосы прокрутки                                      | Обычно выключают для чистого дизайна              |
| `alwaysBounceVertical` / `alwaysBounceHorizontal`                 | `Bool` (false)                                             | Всегда отскок даже если контент меньше экрана                    | `.vertical = true` — стандартный «iOS-отскок»     |
| `isPagingEnabled`                                                 | `Bool` (false)                                             | Включить постраничную прокрутку (как в [[UIPageViewController]]) | Карусели изображений, onboarding                  |
| `isScrollEnabled`                                                 | `Bool` (true)                                              | Включить/выключить прокрутку                                     | Отключение при полноэкранных модалках             |
| `zoomScale` / `minimumZoomScale` / `maximumZoomScale`             | [[CGFloat]] (1.0 / 1.0 / 1.0)                              | Масштабирование контента (pinch to zoom)                         | Просмотр фото, PDF, карт                          |
| `keyboardDismissMode`                                             | `UIScrollView.KeyboardDismissMode` (.none)                 | Как скрывать клавиатуру при скролле                              | `.interactive` — стандарт для форм                |
| `refreshControl`                                                  | [[UIRefreshControl]]`?`                                    | Pull-to-refresh (с iOS 10+)                                      | Обновление ленты                                  |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UIScrollView + [[Auto Layout]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

class FeedViewController: UIViewController {
    
    private let viewModel = FeedViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var scrollView: UIScrollView = {
        let sv = UIScrollView()
        sv.alwaysBounceVertical = true
        sv.showsVerticalScrollIndicator = false
        sv.keyboardDismissMode = .interactive
        sv.translatesAutoresizingMaskIntoConstraints = false
        return sv
    }()
    
    private lazy var contentStack: UIStackView = {
        let stack = UIStackView()
        stack.axis = .vertical
        stack.spacing = 16
        stack.alignment = .fill
        stack.distribution = .fill
        stack.translatesAutoresizingMaskIntoConstraints = false
        return stack
    }()
    
    private lazy var refreshControl: UIRefreshControl = {
        let rc = UIRefreshControl()
        rc.addTarget(self, action: #selector(refresh), for: .valueChanged)
        return rc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        scrollView.addSubview(contentStack)
        scrollView.refreshControl = refreshControl
        
        view.addSubview(scrollView)
        
        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: view.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            
            contentStack.topAnchor.constraint(equalTo: scrollView.contentLayoutGuide.topAnchor, constant: 16),
            contentStack.bottomAnchor.constraint(equalTo: scrollView.contentLayoutGuide.bottomAnchor, constant: -16),
            contentStack.leadingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.leadingAnchor, constant: 16),
            contentStack.trailingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.trailingAnchor, constant: -16),
            contentStack.widthAnchor.constraint(equalTo: scrollView.frameLayoutGuide.widthAnchor, constant: -32)
        ])
    }
    
    private func bindViewModel() {
        viewModel.$items
            .receive(on: DispatchQueue.main)
            .sink { [weak self] items in
                self?.updateContent(with: items)
                self?.refreshControl.endRefreshing()
            }
            .store(in: &cancellables)
    }
    
    private func updateContent(with items: [FeedItem]) {
        contentStack.arrangedSubviews.forEach { $0.removeFromSuperview() }
        
        for item in items {
            let card = FeedItemCardView(item: item)
            contentStack.addArrangedSubview(card)
        }
    }
    
    @objc private func refresh() {
        viewModel.refresh()
    }
}

// ViewModel (пример)
@MainActor
class FeedViewModel: ObservableObject {
    @Published var items: [FeedItem] = []
    
    func refresh() {
        // имитация загрузки
        Task {
            try? await Task.sleep(nanoseconds: 1_000_000_000)
            items = (1...10).map { FeedItem(title: "Элемент \($0)") }
        }
    }
}
```

### Лучшие практики UIScrollView в 2026 году

- **Всегда** задавайте `contentSize` через Auto Layout (через contentLayoutGuide) — не вручную  
- **Используйте** `contentInsetAdjustmentBehavior = .automatic` — автоматически учитывает safe area и navigation/tab bar  
- **Для pull-to-refresh** — добавляйте `UIRefreshControl` — это стандарт 2026  
- **Для масштабирования** — задавайте `minimumZoomScale` / `maximumZoomScale` и реализуйте `viewForZooming(in:)`  
- **Для производительности** — используйте `contentInset` вместо больших отступов в subviews  
- **Для SwiftUI** — используйте `ScrollView` — UIScrollView нужен только в UIKit или смешанных проектах  
- **Для Combine** — подписывайтесь на `publisher(for: \.contentOffset)` или `publisher(for: \.contentSize)`  
- **Документируйте** — пишите комментарий:

```swift
/// UIScrollView с вертикальным стеком контента и pull-to-refresh
private lazy var scrollView: UIScrollView = {
    let sv = UIScrollView()
    sv.alwaysBounceVertical = true
    sv.keyboardDismissMode = .interactive
    return sv
}()
```

**Короткий итог 2026**:
> `UIScrollView` — это **контейнер с прокруткой**, который лежит в основе всех скроллируемых интерфейсов UIKit.  
> В 2026 году:  
> - ключевые свойства — `contentSize`, `contentOffset`, `contentInset`, `refreshControl`  
> - обязательно используйте Auto Layout + `contentLayoutGuide` для `contentSize`  
> - идеален для лент, профилей, форм, галерей, полноэкранного контента  
> - в SwiftUI — заменяется на `ScrollView`  
> - это **фундаментальный** компонент, без которого не обходится почти ни один экран  

Удачи с плавными, адаптивными и производительными прокрутками в твоём приложении! 📜