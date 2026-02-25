**UIPageControl** — это встроенный индикатор страниц в [[UIKit]], который показывает пользователю:

- сколько всего страниц (точек)
- какая страница сейчас активна (выделенная точка)

Это классический «точечный» индикатор, который почти всегда используется вместе с **UIPageViewController**, **UIScrollView** с paging или **TabView.page** в [[SwiftUI]].

В 2026 году UIPageControl остаётся **стандартным** и **самым часто используемым** индикатором постраничной навигации в UIKit-приложениях.

### Основные свойства UIPageControl (самые важные в 2026)

| Свойство                            | Тип / Значение по умолчанию                                                    | Что делает / зачем нужно                     | Самый частый сценарий                    |
| ----------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------- | ---------------------------------------- |
| `numberOfPages`                     | [[Int]] (0)                                                                    | Общее количество страниц (точек)             | Обязательно задавать                     |
| `currentPage`                       | `Int` (0)                                                                      | Индекс текущей активной страницы             | Синхронизация с UIPageViewController     |
| `currentPageIndicatorTintColor`     | [[UIColor]]`?` (системный синий)                                               | Цвет выделенной (активной) точки             | Кастомизация под бренд                   |
| `pageIndicatorTintColor`            | `UIColor?` (серый)                                                             | Цвет неактивных точек                        | Контраст с currentPageIndicatorTintColor |
| `hidesForSinglePage`                | [[Bool]] (false)                                                               | Скрывать индикатор, если всего одна страница | Очень рекомендуется включать             |
| `backgroundStyle` (iOS 14+)         | `UIPageControl.BackgroundStyle` (.automatic / .prominent / .minimal / .custom) | Стиль фона (с iOS 14+)                       | `.automatic` — стандарт                  |
| `preferredIndicatorImage` (iOS 14+) | [[UIImage]]`?`                                                                 | Кастомная картинка вместо точки (для всех)   | Редко, для брендированных точек          |
| `setIndicatorImage(_:forPage:)`     | `Void`                                                                         | Разная картинка для конкретной страницы      | Очень редко                              |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UIPageViewController]] + UIPageControl + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

class OnboardingViewController: UIPageViewController {
    
    private let viewModel = OnboardingViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var pageControl: UIPageControl = {
        let pc = UIPageControl()
        pc.numberOfPages = viewModel.pages.count
        pc.currentPage = 0
        pc.currentPageIndicatorTintColor = .systemBlue
        pc.pageIndicatorTintColor = .systemGray
        pc.hidesForSinglePage = true
        pc.backgroundStyle = .prominent  // современный стиль с iOS 14+
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
        
        // Начальная страница
        if let firstVC = viewModel.pages.first {
            setViewControllers([firstVC], direction: .forward, animated: true)
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

// ViewModel (пример)
@MainActor
class OnboardingViewModel: ObservableObject {
    let pages: [UIViewController] = [
        OnboardingPageVC(image: "page1", title: "Добро пожаловать!"),
        OnboardingPageVC(image: "page2", title: "Функции"),
        OnboardingPageVC(image: "page3", title: "Начнём!")
    ]
    
    @Published var currentPageIndex: Int = 0
}
```

### Современные альтернативы UIPageControl в 2026 году

| Компонент / Способ                           | Когда лучше UIPageControl        | Минимальная версия | Плюсы                                             |
| -------------------------------------------- | -------------------------------- | ------------------ | ------------------------------------------------- |
| **TabView + .tabViewStyle(.page)** в SwiftUI | Новый проект или SwiftUI-экран   | iOS 14+            | Встроенный индикатор страниц, проще синхронизация |
| **UIPageControl + [[UIPageViewController]]** | UIKit + классический онбординг   | iOS 5+             | Полный контроль, кастомизация                     |
| **Custom dots ([[UIView]] + CALayer)**       | Очень специфический дизайн точек | iOS 13+            | Полная кастомизация (анимация, градиент)          |
| **PageControl в [[SwiftUI]]**                | SwiftUI + кастомный индикатор    | iOS 14+            | `.pageIndicator` стиль                            |

### Лучшие практики UIPageControl в 2026 году

- **Всегда** задавайте `numberOfPages` и `currentPage`  
- **Включайте** `hidesForSinglePage = true` — если одна страница, индикатор не нужен  
- **Используйте** `currentPageIndicatorTintColor` и `pageIndicatorTintColor` для кастомизации  
- **Для Combine** — синхронизируйте `currentPage` через `@Published` или `publisher(for:)`  
- **Для доступности** — задавайте `accessibilityLabel = "Страница \(currentPage + 1) из \(numberOfPages)"`  
- **Для SwiftUI** — используйте встроенный индикатор в `TabView(.page)` — UIPageControl нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UIPageControl для отображения текущей страницы онбординга
private lazy var pageControl: UIPageControl = {
    let pc = UIPageControl()
    pc.numberOfPages = viewModel.pages.count
    pc.currentPageIndicatorTintColor = .systemBlue
    pc.pageIndicatorTintColor = .systemGray
    pc.hidesForSinglePage = true
    return pc
}()
```

**Короткий итог 2026**:
> `UIPageControl` — классический индикатор страниц в виде точек.  
> В 2026 году:  
> - ключевые свойства — `numberOfPages`, `currentPage`, `currentPageIndicatorTintColor`  
> - почти всегда используется вместе с `UIPageViewController`  
> - идеален для онбординга, галереи изображений, wizard-экранов  
> - в SwiftUI — встроен в `TabView` со стилем `.page`  
> - это **надёжный** и **стандартный** способ показать пользователю, на какой странице он находится  
