**UIProgressView** — это встроенный индикатор прогресса в [[UIKit]], который показывает степень выполнения задачи в виде горизонтальной полосы (прогресс-бара).

Это классический и до сих пор один из самых часто используемых компонентов для отображения прогресса загрузки, обработки файлов, установки, таймера и т.д.

### Основные свойства UIProgressView (актуально на 2026 год)

| Свойство                       | Тип / Значение по умолчанию              | Что делает / зачем нужно                                                   | Самый частый сценарий                                |
| ------------------------------ | ---------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------- |
| `progress`                     | [[Float]] (0.0...1.0)                    | Текущий прогресс (0.0 = начало, 1.0 = конец)                               | Основное свойство, к которому привязывается прогресс |
| `progressTintColor`            | [[UIColor]]`?` (системный синий)         | Цвет заполненной части прогресс-бара                                       | Кастомизация под бренд                               |
| `trackTintColor`               | `UIColor?` (серый)                       | Цвет незаполненной (фоновой) части                                         | Контраст с progressTintColor                         |
| `progressViewStyle`            | `UIProgressView.Style` (.default / .bar) | Стиль: .default — округлённый, .bar — прямоугольный (как в navigation bar) | .default — в 90% случаев                             |
| `progressImage` / `trackImage` | [[UIImage]]`?`                           | Кастомное изображение вместо цвета                                         | Редко, для очень специфического дизайна              |
| `observedProgress` (iOS 9+)    | `Progress?`                              | Автоматическая привязка к объекту `Progress` (с iOS 9)                     | Современный способ привязки прогресса                |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UITableView]] / [[UIViewController]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

class DownloadViewController: UIViewController {
    
    private let viewModel = DownloadViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var progressView: UIProgressView = {
        let pv = UIProgressView(progressViewStyle: .default)
        pv.progressTintColor = .systemBlue
        pv.trackTintColor = .systemGray5
        pv.layer.cornerRadius = 8
        pv.clipsToBounds = true
        pv.translatesAutoresizingMaskIntoConstraints = false
        return pv
    }()
    
    private lazy var statusLabel: UILabel = {
        let lbl = UILabel()
        lbl.textAlignment = .center
        lbl.font = .systemFont(ofSize: 16)
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Загрузка файла"
        view.backgroundColor = .systemBackground
        
        view.addSubview(progressView)
        view.addSubview(statusLabel)
        
        NSLayoutConstraint.activate([
            progressView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            progressView.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -40),
            progressView.widthAnchor.constraint(equalToConstant: 300),
            progressView.heightAnchor.constraint(equalToConstant: 16),
            
            statusLabel.topAnchor.constraint(equalTo: progressView.bottomAnchor, constant: 20),
            statusLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        bindViewModel()
        
        // Запускаем загрузку
        viewModel.startDownload()
    }
    
    private func bindViewModel() {
        // Привязка прогресса
        viewModel.$progress
            .receive(on: DispatchQueue.main)
            .assign(to: \.progress, on: progressView)
            .store(in: &cancellables)
        
        // Привязка статуса
        viewModel.$statusMessage
            .receive(on: DispatchQueue.main)
            .assign(to: \.text, on: statusLabel)
            .store(in: &cancellables)
        
        // Реакция на завершение
        viewModel.$isCompleted
            .receive(on: DispatchQueue.main)
            .sink { [weak self] completed in
                if completed {
                    self?.statusLabel.text = "Загрузка завершена!"
                    self?.progressView.progressTintColor = .systemGreen
                }
            }
            .store(in: &cancellables)
    }
}

// ViewModel (пример)
@MainActor
class DownloadViewModel: ObservableObject {
    @Published var progress: Float = 0.0
    @Published var statusMessage: String = "Ожидание..."
    @Published var isCompleted = false
    
    private var cancellables = Set<AnyCancellable>()
    
    func startDownload() {
        statusMessage = "Загрузка началась..."
        
        // Имитация загрузки
        Timer.publish(every: 0.1, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in
                guard let self else { return }
                
                if self.progress < 1.0 {
                    self.progress += 0.01
                    self.statusMessage = String(format: "Прогресс: %.0f%%", self.progress * 100)
                } else {
                    self.isCompleted = true
                    self.statusMessage = "Готово!"
                }
            }
            .store(in: &cancellables)
    }
}
```

### Современные альтернативы UIProgressView в 2026 году

| Компонент / Способ                  | Когда лучше UIProgressView                           | Минимальная версия | Плюсы                                                            |
| ----------------------------------- | ---------------------------------------------------- | ------------------ | ---------------------------------------------------------------- |
| **ProgressView** в [[SwiftUI]]      | Новый проект или SwiftUI-экран                       | iOS 14+            | Декларативный стиль, `.progressViewStyle(.linear)` / `.circular` |
| **UIProgressView + [[Combine]]**    | UIKit + MVVM                                         | iOS 9+             | Реактивная привязка через `.assign(to:)`                         |
| **Custom [[UIView]] + [[CALayer]]** | Очень специфический дизайн (градиент, анимированный) | iOS 13+            | Полная кастомизация                                              |
| **[[UIActivityIndicatorView]]**     | Когда прогресс неизвестен (indeterminate)            | iOS 13+            | Простой спиннер вместо полосы                                    |
| **Progress** + `observedProgress`   | Когда есть объект `Progress` (например, FileManager) | iOS 9+             | Автоматическая привязка                                          |

### Лучшие практики UIProgressView в 2026 году

- **Всегда** устанавливайте `progress` в диапазоне 0.0...1.0  
- **Используйте** `progressTintColor` и `trackTintColor` для кастомизации  
- **Для анимации** — применяйте `UIView.animate(withDuration:)` при изменении `progress`  
- **Для Combine / async** — привязывайте `progress` через `.assign(to: \.progress, on: progressView)`  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue` (например, "Загрузка: 45%")  
- **Для SwiftUI** — используйте `ProgressView` — UIProgressView нужен только в UIKit  
- **Для indeterminate прогресса** — лучше `UIActivityIndicatorView`  
- **Документируйте** — пишите комментарий:

```swift
/// UIProgressView для отображения прогресса загрузки файла
private lazy var progressView: UIProgressView = {
    let pv = UIProgressView(progressViewStyle: .default)
    pv.progressTintColor = .systemBlue
    pv.trackTintColor = .systemGray5
    return pv
}()
```

**Короткий итог 2026**:
> `UIProgressView` — классический индикатор прогресса в виде полосы (0.0...1.0).  
> В 2026 году:  
> - ключевые свойства — `progress`, `progressTintColor`, `trackTintColor`  
> - идеален для загрузки файлов, установки, обработки данных  
> - в SwiftUI — заменяется на `ProgressView`  
> - в UIKit — синхронизируется с `@Published` через Combine или прямое присваивание  
> - это **надёжный** и **стандартный** способ показать прогресс выполнения  
