**UIActivityIndicatorView** — это встроенный индикатор загрузки (спиннер) в UIKit, который показывает пользователю, что приложение выполняет какую-то длительную операцию и просит подождать.

Это один из самых часто используемых элементов интерфейса в iOS-приложениях, особенно в 2026 году, когда пользователи ожидают мгновенной обратной связи даже при сетевых запросах, загрузке данных или обработке.

### Основные характеристики (актуально на 2026 год)

| Характеристика                     | Описание                                                                 | Значение по умолчанию / Рекомендация 2026 |
|------------------------------------|--------------------------------------------------------------------------|--------------------------------------------|
| **Стиль (style)**                  | `.medium`, `.large` (ранее `.gray`, `.white`, `.whiteLarge`)             | `.medium` — самый универсальный             |
| **Цвет (color)**                   | Можно менять `.tintColor` или `.color`                                   | `.systemGray` или акцентный цвет приложения |
| **Анимация**                       | Запускается автоматически при `startAnimating()`, останавливается при `stopAnimating()` | Всегда проверяй `isAnimating`              |
| **HidesWhenStopped**               | Скрывает спиннер, когда анимация остановлена                             | `true` — стандарт и лучший UX              |
| **Background / эффект**            | Можно добавить фон, blur, scale-анимацию                                 | Часто оборачивают в контейнер с blur       |
| **SwiftUI аналог**                 | `ProgressView(style: .circular)`                                         | В SwiftUI-проектах — ProgressView           |

### Когда использовать UIActivityIndicatorView в 2026 году

| Сценарий                                      | Где обычно ставят                                   | Рекомендуемый стиль / поведение |
|-----------------------------------------------|-----------------------------------------------------|---------------------------------|
| Загрузка данных из сети (API, Firebase)       | По центру экрана или внутри ячейки                  | `.large` + полупрозрачный фон   |
| Обновление таблицы / коллекции                | Вверху экрана или вместо пустого состояния          | `.medium` + "Обновление..."     |
| Загрузка изображения (placeholder)            | Внутри UIImageView или ячейки                       | `.medium` внутри контейнера     |
| Долгая операция (сохранение, обработка фото)  | По центру модального экрана                         | `.large` + затемнение фона      |
| Pull-to-refresh (встроенный)                  | Уже встроен в UIRefreshControl                      | Не нужен отдельный спиннер      |
| Пагинация (бесконечный скролл)                | Внизу списка                                        | `.medium` в футере              |

### Самый популярный и современный паттерн 2026 года

#### 1. Простой спиннер по центру экрана

```swift
final class LoadingOverlayView: UIView {
    
    private let spinner = UIActivityIndicatorView(style: .large)
    private let backgroundView = UIVisualEffectView(effect: UIBlurEffect(style: .systemMaterial))
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        backgroundColor = .clear
        backgroundView.frame = bounds
        addSubview(backgroundView)
        
        spinner.translatesAutoresizingMaskIntoConstraints = false
        spinner.hidesWhenStopped = true
        spinner.color = .systemBlue
        addSubview(spinner)
        
        NSLayoutConstraint.activate([
            spinner.centerXAnchor.constraint(equalTo: centerXAnchor),
            spinner.centerYAnchor.constraint(equalTo: centerYAnchor)
        ])
    }
    
    func start() {
        spinner.startAnimating()
        isHidden = false
    }
    
    func stop() {
        spinner.stopAnimating()
        isHidden = true
    }
}
```

Использование в контроллере:

```swift
class DataViewController: UIViewController {
    
    private lazy var loadingOverlay = LoadingOverlayView(frame: view.bounds)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(loadingOverlay)
        loadingOverlay.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    }
    
    func fetchData() {
        loadingOverlay.start()
        
        Task {
            do {
                let data = try await api.fetchData()
                await MainActor.run {
                    // обновить UI
                    loadingOverlay.stop()
                }
            } catch {
                await MainActor.run {
                    loadingOverlay.stop()
                    showError(error)
                }
            }
        }
    }
}
```

#### 2. Спиннер внутри ячейки таблицы / коллекции

```swift
final class LoadingCell: UITableViewCell {
    
    private let spinner = UIActivityIndicatorView(style: .medium)
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setup()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setup() {
        spinner.translatesAutoresizingMaskIntoConstraints = false
        spinner.hidesWhenStopped = true
        contentView.addSubview(spinner)
        
        NSLayoutConstraint.activate([
            spinner.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            spinner.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }
    
    func startLoading() {
        spinner.startAnimating()
    }
    
    func stopLoading() {
        spinner.stopAnimating()
    }
}
```

### Лучшие практики UIActivityIndicatorView в Swift 2026

- **Всегда** используй `hidesWhenStopped = true` — стандартный и лучший UX  
- **Стиль** — `.large` для полноэкранной загрузки, `.medium` для ячеек/локальных зон  
- **color** — подстраивай под акцентный цвет приложения или `.systemGray`  
- **Оборачивай в blur** — UIVisualEffectView + spinner выглядит современно и профессионально  
- **Не забывай `stopAnimating()`** — особенно в `viewWillDisappear` / `sceneDidEnterBackground`  
- **Асинхронность** — запускай в `Task` и останавливай на `@MainActor`  
- **SwiftUI аналог** — `ProgressView(style: .circular)` — в чистых SwiftUI-проектах лучше его  
- **Документируйте** — пиши комментарий «UIActivityIndicatorView — индикатор загрузки данных с блюром»

**Короткий девиз 2026**:
> UIActivityIndicatorView — это **классический и до сих пор идеальный** способ показать пользователю «подожди, идёт загрузка».  
> В 2026 году его ставят везде: по центру экрана с блюром, в ячейках, в футере списка.  
> Всегда `hidesWhenStopped = true`, `start` перед асинхронной задачей, `stop` после.

Удачи с приятными и информативными состояниями загрузки в твоём приложении! ⏳