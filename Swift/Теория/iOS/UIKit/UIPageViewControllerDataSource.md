**UIPageViewControllerDataSource** — это протокол в [[UIKit]], который отвечает за **поставку страниц** (view controllers) для **[[UIPageViewController]]**.

Он определяет:
- сколько всего страниц,
- какой контроллер показывать **перед** текущим,
- какой контроллер показывать **после** текущего,
- сколько страниц отображать в индикаторе (page control).

Без реализации этого протокола UIPageViewController **не сможет перелистывать страницы**.

### Основные методы протокола (все обязательные)

| Метод делегата                                | Что возвращает          | Когда вызывается                                             | Самый частый сценарий          |
| --------------------------------------------- | ----------------------- | ------------------------------------------------------------ | ------------------------------ |
| `pageViewController(_:viewControllerBefore:)` | [[UIViewController]]`?` | Когда пользователь пытается пролистать **влево** / **вверх** | Возвращаем предыдущую страницу |
| `pageViewController(_:viewControllerAfter:)`  | `UIViewController?`     | Когда пользователь пытается пролистать **вправо** / **вниз** | Возвращаем следующую страницу  |
| `presentationCount(for:)`                     | [[Int]]                 | Для отображения количества точек в [[UIPageControl]]         | `pages.count`                  |
| `presentationIndex(for:)`                     | `Int`                   | Какой индекс текущей страницы показывать в page control      | Текущий индекс                 |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UIPageViewController]] + [[UIPageViewControllerDataSource|DataSource]] + [[Delegate]] + [[UIPageControl]] + [[Combine]])

```swift
import UIKit
import Combine

class OnboardingViewController: UIPageViewController {
    
    private let viewModel = OnboardingViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var pageControl: UIPageControl = {
        let pc = UIPageControl()
        pc.numberOfPages = viewModel.pages.count
        pc.currentPageIndicatorTintColor = .systemBlue
        pc.pageIndicatorTintColor = .systemGray
        pc.hidesForSinglePage = true
        pc.translatesAutoresizingMaskIntoConstraints = false
        return pc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        dataSource = self
        delegate = self  // обязательно для обновления pageControl
        
        view.backgroundColor = .systemBackground
        
        // Добавляем индикатор страниц
        view.addSubview(pageControl)
        NSLayoutConstraint.activate([
            pageControl.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            pageControl.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        // Устанавливаем первую страницу
        if let firstPage = viewModel.pages.first {
            setViewControllers([firstPage], direction: .forward, animated: true)
            pageControl.currentPage = 0
        }
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$currentPageIndex
            .receive(on: DispatchQueue.main)
            .assign(to: \.currentPage, on: pageControl)
            .store(in: &cancellables)
    }
}

// MARK: - UIPageViewControllerDataSource (обязательно)
extension OnboardingViewController: UIPageViewControllerDataSource {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerBefore viewController: UIViewController) -> UIViewController? {
        
        guard let currentIndex = viewModel.pages.firstIndex(of: viewController),
              currentIndex > 0 else {
            return nil
        }
        
        return viewModel.pages[currentIndex - 1]
    }
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerAfter viewController: UIViewController) -> UIViewController? {
        
        guard let currentIndex = viewModel.pages.firstIndex(of: viewController),
              currentIndex < viewModel.pages.count - 1 else {
            return nil
        }
        
        return viewModel.pages[currentIndex + 1]
    }
    
    // Количество точек в индикаторе
    func presentationCount(for pageViewController: UIPageViewController) -> Int {
        viewModel.pages.count
    }
    
    // Текущий индекс для индикатора
    func presentationIndex(for pageViewController: UIPageViewController) -> Int {
        viewModel.currentPageIndex
    }
}

// MARK: - UIPageViewControllerDelegate (рекомендуется)
extension OnboardingViewController: UIPageViewControllerDelegate {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            didFinishAnimating finished: Bool,
                            previousViewControllers: [UIViewController],
                            transitionCompleted completed: Bool) {
        guard completed,
              let currentVC = pageViewController.viewControllers?.first,
              let newIndex = viewModel.pages.firstIndex(of: currentVC) else {
            return
        }
        
        viewModel.currentPageIndex = newIndex
    }
}

// ViewModel (пример)
@MainActor
class OnboardingViewModel: ObservableObject {
    let pages: [UIViewController] = [
        OnboardingPageVC(imageName: "welcome", title: "Добро пожаловать"),
        OnboardingPageVC(imageName: "features", title: "Возможности"),
        OnboardingPageVC(imageName: "start", title: "Начнём!")
    ]
    
    @Published var currentPageIndex: Int = 0
}
```

### Лучшие практики UIPageViewControllerDataSource в 2026 году

- **Всегда** реализуйте все четыре метода — без них перелистывание не работает  
- **Кэшируйте** страницы — создавайте их один раз и храните в массиве (не создавайте заново при каждом вызове)  
- **Используйте** `===` или `is` для сравнения view controllers вместо `==` (они разные экземпляры)  
- **Синхронизируйте** `presentationIndex` через `@Published` + `Combine` или `viewModel.currentPageIndex`  
- **Для бесконечной карусели** — возвращайте первую страницу после последней и наоборот  
- **Для SwiftUI** — используйте `TabView` со стилем `.page` — UIPageViewControllerDataSource нужен только в UIKit  
- **Для доступности** — синхронизируйте `accessibilityLabel` page control с текущей страницей  
- **Документируйте** — пишите комментарий:

```swift
/// Поставляет страницы для UIPageViewController (онбординг)
extension OnboardingViewController: UIPageViewControllerDataSource {
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerBefore viewController: UIViewController) -> UIViewController? {
        // ...
    }
}
```

**Короткий итог 2026**:
> `UIPageViewControllerDataSource` — протокол, который **поставляет страницы** для постраничной навигации в UIPageViewController.  
> В 2026 году:  
> - ключевые методы — `viewControllerBefore`, `viewControllerAfter`, `presentationCount`, `presentationIndex`  
> - самый важный сценарий — онбординг, галерея изображений, wizard-экранов  
> - всегда работает в паре с `UIPageViewControllerDelegate` (для обновления состояния)  
> - в SwiftUI — заменяется на `TabView` с `.page` стилем  
> - это **обязательный** протокол для любой карусели страниц в UIKit  
