**UIPageViewController** — это контроллер в [[UIKit]], предназначенный для создания **постраничной (page-based) навигации**, где пользователь может **перелистывать страницы** (свайпом влево/вправо или вверх/вниз), как в книге, презентации или онбординге.

Это классический компонент, который до сих пор активно используется в iOS-приложениях, хотя в новых проектах на [[SwiftUI]] чаще применяют `TabView` с `.tabViewStyle(.page)`.

### Основные возможности UIPageViewController (актуально на 2026 год)

| Свойство / Метод                                       | Тип / Значение по умолчанию                                            | Что делает / зачем нужно                                 | Самый частый сценарий                   |
| ------------------------------------------------------ | ---------------------------------------------------------------------- | -------------------------------------------------------- | --------------------------------------- |
| `transitionStyle`                                      | `UIPageViewController.TransitionStyle` (.scroll / .pageCurl)           | Стиль анимации перехода                                  | `.scroll` — современный и рекомендуемый |
| `navigationOrientation`                                | `UIPageViewController.NavigationOrientation` (.horizontal / .vertical) | Направление свайпа (горизонтально или вертикально)       | `.horizontal` — 95% случаев             |
| `dataSource`                                           | [[UIPageViewControllerDataSource]]`?`                                  | Источник страниц (какой контроллер следующий/предыдущий) | Обязательно задавать                    |
| `delegate`                                             | [[UIPageViewControllerDelegate]]`?`                                    | Реакция на смену страницы, завершение анимации           | Отслеживание текущей страницы           |
| `setViewControllers(_:direction:animated:completion:)` | `Void`                                                                 | Установить начальные контроллеры или перейти к новому    | Установка стартовой страницы            |
| `viewControllers`                                      | `[UIViewController]?` (только чтение)                                  | Текущие отображаемые контроллеры (обычно 1)              | Получение текущей страницы              |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UIPageViewController + [[UIKit]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

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
        pc.translatesAutoresizingMaskIntoConstraints = false
        return pc
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        dataSource = self
        delegate = self
        
        // Начальная страница
        if let firstVC = viewModel.pages.first {
            setViewControllers([firstVC], direction: .forward, animated: true)
        }
        
        view.backgroundColor = .systemBackground
        
        // Добавляем page control внизу
        view.addSubview(pageControl)
        NSLayoutConstraint.activate([
            pageControl.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            pageControl.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$currentPageIndex
            .receive(on: DispatchQueue.main)
            .sink { [weak self] index in
                self?.pageControl.currentPage = index
            }
            .store(in: &cancellables)
    }
}

// MARK: - UIPageViewControllerDataSource
extension OnboardingViewController: UIPageViewControllerDataSource {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerBefore viewController: UIViewController) -> UIViewController? {
        guard let currentIndex = viewModel.pages.firstIndex(where: { $0 === viewController }),
              currentIndex > 0 else { return nil }
        return viewModel.pages[currentIndex - 1]
    }
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            viewControllerAfter viewController: UIViewController) -> UIViewController? {
        guard let currentIndex = viewModel.pages.firstIndex(where: { $0 === viewController }),
              currentIndex < viewModel.pages.count - 1 else { return nil }
        return viewModel.pages[currentIndex + 1]
    }
    
    func presentationCount(for pageViewController: UIPageViewController) -> Int {
        viewModel.pages.count
    }
    
    func presentationIndex(for pageViewController: UIPageViewController) -> Int {
        viewModel.currentPageIndex
    }
}

// MARK: - UIPageViewControllerDelegate
extension OnboardingViewController: UIPageViewControllerDelegate {
    
    func pageViewController(_ pageViewController: UIPageViewController,
                            didFinishAnimating finished: Bool,
                            previousViewControllers: [UIViewController],
                            transitionCompleted completed: Bool) {
        guard completed,
              let currentVC = pageViewController.viewControllers?.first,
              let index = viewModel.pages.firstIndex(where: { $0 === currentVC }) else { return }
        
        viewModel.currentPageIndex = index
    }
}

// ViewModel
@MainActor
class OnboardingViewModel: ObservableObject {
    let pages: [UIViewController] = [
        OnboardingPageVC(image: "page1", title: "Добро пожаловать!", subtitle: "Первая страница онбординга"),
        OnboardingPageVC(image: "page2", title: "Функции", subtitle: "Второй экран с описанием"),
        OnboardingPageVC(image: "page3", title: "Начнём!", subtitle: "Последняя страница")
    ]
    
    @Published var currentPageIndex: Int = 0
}

// Пример страницы онбординга
class OnboardingPageVC: UIViewController {
    let imageName: String
    let titleText: String
    let subtitleText: String
    
    init(image: String, title: String, subtitle: String) {
        self.imageName = image
        self.titleText = title
        self.subtitleText = subtitle
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Настройка UI страницы (image, title, subtitle)
        // ...
    }
}
```

### Современные альтернативы UIPageViewController в 2026 году

| Компонент / Способ                           | Когда лучше UIPageViewController         | Минимальная версия | Плюсы                                |
| -------------------------------------------- | ---------------------------------------- | ------------------ | ------------------------------------ |
| **TabView + .tabViewStyle(.page)** в SwiftUI | Новый проект или SwiftUI-экран           | iOS 14+            | Декларативный стиль, проще анимация  |
| **UIPageViewController**                     | UIKit + сложная кастомизация страниц     | iOS 5+             | Полный контроль над transition       |
| **[[UIScrollView]] + pagingEnabled**         | Когда нужна очень кастомная карусель     | iOS 13+            | Максимальная гибкость                |
| **[[UICollectionView]] + horizontal flow**   | Карусель с динамическим контентом        | iOS 13+            | Поддержка diffable data source       |
| **[[SwiftUI]] TabView + .page**              | SwiftUI + онбординг, галерея изображений | iOS 14+            | Лучшая производительность и анимация |

### Лучшие практики UIPageViewController в 2026 году

- **Всегда** реализуйте `dataSource` и `delegate` — без них ничего не будет работать  
- **Используйте** `.scroll` стиль — `.pageCurl` выглядит устаревшим  
- **Для индикатора страниц** — добавляйте `UIPageControl` вручную (встроенный индикатор в UIPageViewController устарел)  
- **Для Combine** — используйте `currentPageIndex` через `@Published` или `publisher(for:)`  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint` для каждой страницы  
- **Для SwiftUI** — используйте `TabView` со стилем `.page` — UIPageViewController нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UIPageViewController для онбординга с 3 страницами
class OnboardingViewController: UIPageViewController {
    // ...
}
```

**Короткий итог 2026**:
> `UIPageViewController` — контроллер для **постраничной навигации** (свайп влево/вправо между экранами).  
> В 2026 году:  
> - ключевые протоколы — `dataSource`, `delegate`  
> - идеален для онбординга, галереи изображений, wizard-экранов  
> - в SwiftUI — заменяется на `TabView` с `.tabViewStyle(.page)`  
> - в UIKit — это **стандартный** и **надёжный** способ сделать карусель страниц  
