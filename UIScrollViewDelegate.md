**`UIScrollViewDelegate`** — это протокол в [[UIKit]], который определяет методы-делегаты для отслеживания и управления поведением прокрутки в **[[UIScrollView]]** (и всех его наследниках: [[UITableView]], [[UICollectionView]], [[UITextView]], [[MKMapView]], [[WKWebView]] и т.д.).

Это **самый важный** протокол для кастомизации прокрутки, бесконечной прокрутки, parallax-эффектов, sticky-заголовков, pull-to-refresh и многих других типичных сценариев iOS-приложений.

### Основные методы протокола UIScrollViewDelegate (актуально на 2026 год)

| Метод делегата                                      | Когда вызывается                                                                 | Самый частый сценарий использования в 2026 | Важные замечания |
|-----------------------------------------------------|----------------------------------------------------------------------------------|---------------------------------------------|------------------|
| `scrollViewDidScroll(_:)`                           | При любом изменении позиции контента (самый часто вызываемый метод)              | Parallax, sticky headers, infinite scroll, custom animations | Вызывается очень часто — оптимизируйте! |
| `scrollViewWillBeginDragging(_:)`                   | Пользователь начал прокрутку пальцем                                             | Отслеживание начала взаимодействия          | — |
| `scrollViewDidEndDragging(_:willDecelerate:)`       | Пользователь убрал палец (с параметром, будет ли инерция)                        | Запуск/остановка дополнительных анимаций    | `willDecelerate` — ключевой флаг |
| `scrollViewDidEndDecelerating(_:)`                  | Прокрутка с инерцией полностью остановилась                                      | Загрузка следующей страницы (infinite scroll) | Самый надёжный момент "конца прокрутки" |
| `scrollViewDidEndScrollingAnimation(_:)`            | Закончилась программная анимация прокрутки (`setContentOffset(animated:)`)       | Обновление UI после анимированного скролла | — |
| `scrollViewWillBeginDecelerating(_:)`               | Начинается замедление после инерции                                              | Редко используется                          | — |
| `scrollViewShouldScrollToTop(_:)`                   | Пользователь тапнул по статус-бару                                               | Можно запретить возврат в начало            | Возвращает `Bool` |
| `scrollViewDidZoom(_:)`                             | Изменился масштаб (zoom)                                                         | Обработка зума в галереях/картах            | Для `UIScrollView` с `zoomScale` |
| `viewForZooming(in:)`                               | Возвращает `UIView`, который будет масштабироваться                               | Обязателен для включения зума               | Самый важный для pinch-to-zoom |

### Полный современный пример реализации делегата (рекомендуемый шаблон 2026)

```swift
import UIKit

@MainActor
final class FeedViewController: UIViewController, UIScrollViewDelegate {
    
    private let tableView = UITableView()
    private var isLoadingMore = false
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupTableView()
    }
    
    private func setupTableView() {
        tableView.delegate = self
        tableView.dataSource = self
        tableView.frame = view.bounds
        view.addSubview(tableView)
        
        // Настройки для бесконечной прокрутки
        tableView.contentInset.bottom = 100
    }
    
    // Самый важный метод — отслеживание прокрутки
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let offsetY = scrollView.contentOffset.y
        let contentHeight = scrollView.contentSize.height
        let frameHeight = scrollView.frame.height
        
        // Бесконечная прокрутка: загрузка при достижении низа
        if offsetY + frameHeight >= contentHeight - 200, !isLoadingMore {
            isLoadingMore = true
            loadMoreContent()
        }
        
        // Parallax для заголовка (пример)
        if let header = tableView.tableHeaderView {
            let parallaxFactor: CGFloat = 0.4
            let offset = max(0, -offsetY * parallaxFactor)
            header.frame.origin.y = offset
        }
    }
    
    // Конец инерционной прокрутки — надёжный момент для загрузки
    func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        if isLoadingMore {
            // Можно дополнительно проверить позицию
        }
    }
    
    // Пользователь отпустил палец
    func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
        if !decelerate && isLoadingMore {
            // Если нет инерции, но мы уже в зоне загрузки
            loadMoreContent()
        }
    }
    
    private func loadMoreContent() {
        // Симуляция загрузки
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) { [weak self] in
            guard let self else { return }
            // Добавляем данные
            self.isLoadingMore = false
            self.tableView.reloadData()
        }
    }
}
```

### Лучшие практики UIScrollViewDelegate в 2026

- **Делайте делегат отдельным классом** или extension контроллера — не смешивайте логику  
- **Оптимизируйте** `scrollViewDidScroll` — он вызывается **очень часто** (сотни раз за прокрутку)  
  - избегайте тяжёлых вычислений
  - используйте `CATransaction.disableActions()` для анимаций
- **Для бесконечной прокрутки** — проверяйте позицию в `scrollViewDidScroll` с запасом (threshold ~ 200–300 pt)  
- **Для parallax / sticky headers** — используйте `scrollViewDidScroll` + трансформации/смещение frame  
- **В SwiftUI** — вместо делегата используйте `GeometryReader` + `.coordinateSpace` + `.onPreferenceChange`  
- **Swift 6 strict concurrency** — делегат должен быть `@MainActor` (UIScrollView работает на главном потоке)  
- **Документируйте** — пишите комментарий «scrollViewDidScroll — бесконечная прокрутка + parallax заголовка»

**Короткий итог 2026**:
> `UIScrollViewDelegate` — это **мозг прокрутки** в UIKit.  
> В 2026 году:  
> - главный рабочий метод — `scrollViewDidScroll`  
> - ключевые для UX — `didEndDecelerating` и `didEndDragging`  
> - используйте для: бесконечной прокрутки, parallax, sticky, pull-to-refresh  
> - оптимизируйте — метод вызывается очень часто  
> - в SwiftUI — почти не нужен (заменяется GeometryReader и PreferenceKey)  
