**UIPageViewControllerDelegate** — это протокол в [[UIKit]], который позволяет:

- получать уведомления о смене страницы в **[[UIPageViewController]]**,
- синхронизировать внешние элементы интерфейса (например, [[UIPageControl]]),
- реагировать на завершение анимации перелистывания,
- управлять поведением при programmatic переходах.

Он используется **вместе** с [[UIPageViewControllerDataSource]] (источник данных страниц) и является обязательным для большинства реальных сценариев с постраничной навигацией.

### Основные методы протокола (актуальные в 2026 году)

| Метод делегата                                      | Когда вызывается                                                                 | Что обычно делают внутри метода                                      | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------|
| `pageViewController(_:didFinishAnimating:previousViewControllers:transitionCompleted:)` | После завершения анимации перехода на новую страницу                             | Обновление UIPageControl, сохранение текущего индекса, логирование   | Самый важный метод — синхронизация page control |
| `pageViewController(_:willTransitionTo:)`           | Перед началом анимации перехода (когда пользователь начал листать)              | Подготовка UI, отложенная загрузка данных следующей страницы          | Предзагрузка контента |
| `pageViewController(_:didFinishAnimating:previousViewControllers:transitionCompleted:)` | После завершения анимации (включая отменённые жесты)                            | Проверка `transitionCompleted` → только если переход завершён успешно | Обновление состояния только при успешном переходе |
| `pageViewController(_:spineLocationFor:)`           | При изменении ориентации (редко, только для .pageCurl стиля)                    | Возврат положения «корешка» книги                                     | Почти не используется (стиль .scroll — основной) |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UIPageViewController]] + UIPageViewControllerDelegate + [[UIPageControl]] + [[Combine]])

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
        delegate = self
        
        view.backgroundColor = .systemBackground
        
        // Добавляем page control внизу
        view.addSubview(pageControl)
        NSLayoutConstraint.activate([
            pageControl.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            pageControl.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        // Устанавливаем первую страницу
        if let firstPage = viewModel.pages.first {
            setViewControllers([firstPage], direction: .forward, animated: true)
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

// MARK: - UIPageViewControllerDataSource
extension OnboardingViewController: UIPageViewControllerDataSource {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerBefore viewController: UIViewController) -> UIViewController? {
        guard let currentIndex = viewModel.pages.firstIndex(of: viewController),
              currentIndex > 0 else { return nil }
        return viewModel.pages[currentIndex - 1]
    }
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerAfter viewController: UIViewController) -> UIViewController? {
        guard let currentIndex = viewModel.pages.firstIndex(of: viewController),
              currentIndex < viewModel.pages.count - 1 else { return nil }
        return viewModel.pages[currentIndex + 1]
    }
}

// MARK: - UIPageViewControllerDelegate (самое важное)
extension OnboardingViewController: UIPageViewControllerDelegate {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            didFinishAnimating finished: Bool,
                            previousViewControllers: [UIViewController],
                            transitionCompleted completed: Bool) {
        // Самый важный метод
        guard completed,
              let currentVC = pageViewController.viewControllers?.first,
              let newIndex = viewModel.pages.firstIndex(of: currentVC) else {
            return
        }
        
        viewModel.currentPageIndex = newIndex
    }
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            willTransitionTo pendingViewControllers: [UIViewController]) {
        // Можно начать предзагрузку данных следующей страницы
        if let nextVC = pendingViewControllers.first as? OnboardingPageVC {
            nextVC.startPreloadingContent()
        }
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

### Лучшие практики UIPageViewControllerDelegate в 2026 году

- **Обязательно** реализуйте `didFinishAnimating:previousViewControllers:transitionCompleted:`  
  → именно здесь обновляется UIPageControl и сохраняется текущий индекс  
- **Проверяйте** `transitionCompleted` — если `false`, значит пользователь отменил жест → не обновляйте состояние  
- **Используйте** `willTransitionTo:` для предзагрузки контента (особенно если страницы тяжёлые)  
- **Для кастомной анимации** — комбинируйте с `UIViewControllerTransitioningDelegate` (редко)  
- **Для SwiftUI** — используйте `TabView` со стилем `.page` — UIPageViewControllerDelegate нужен только в UIKit  
- **Для доступности** — синхронизируйте `accessibilityLabel` page control с текущей страницей  
- **Документируйте** — пишите комментарий:

```swift
/// Обновляем текущий индекс страницы и page control после завершения перехода
func pageViewController(_ pageViewController: UIPageViewController,
                        didFinishAnimating finished: Bool,
                        previousViewControllers: [UIViewController],
                        transitionCompleted completed: Bool) {
    // ...
}
```

**Короткий итог 2026**:
> `UIPageViewControllerDelegate` — протокол для **управления поведением** постраничной навигации в UIPageViewController.  
> В 2026 году:  
> - ключевой метод — `didFinishAnimating:previousViewControllers:transitionCompleted:`  
> - самый важный сценарий — синхронизация UIPageControl с текущей страницей  
> - часто используется вместе с `UIPageViewControllerDataSource`  
> - идеален для онбординга, галереи изображений, wizard-экранов  
> - в SwiftUI — эквивалент `TabView` с `.page` стилем  
