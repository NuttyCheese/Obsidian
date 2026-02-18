**Delegate** — это один из самых важных и часто используемых паттернов проектирования в iOS-разработке на Swift/UIKit.

Он позволяет **одному объекту (делегатору)** передавать ответственность за выполнение определённых действий **другому объекту (делегату)**, сохраняя **слабую связь** между ними.

> Проще говоря: Delegate = «когда объект говорит другому: "если что-то произойдёт — сообщи мне, я решу, что делать"».

### 1. Почему delegate — это основа UIKit (2026 реальность)

| Компонент UIKit                  | Зачем нужен delegate                                      | Самый частый делегат в 2026 |
|----------------------------------|------------------------------------------------------------|-----------------------------|
| `UITableView` / `UICollectionView` | Обработка данных, размеров ячеек, действий пользователя    | `UITableViewDelegate`, `UITableViewDataSource` |
| `UITextField` / `UITextView`     | Реакция на ввод, фокус, возврат клавиатуры                 | `UITextFieldDelegate`       |
| `UIScrollView`                   | Отслеживание скролла, зума, начала/окончания перетаскивания | `UIScrollViewDelegate`      |
| `CLLocationManager`              | Получение обновлений геолокации, авторизация               | `CLLocationManagerDelegate` |
| `UNUserNotificationCenter`       | Обработка уведомлений, действий пользователя               | `UNUserNotificationCenterDelegate` |
| `UIImagePickerController`        | Выбор фото/видео из галереи/камеры                         | `UIImagePickerControllerDelegate` |
| `MFMailComposeViewController`    | Отправка почты из приложения                               | `MFMailComposeViewControllerDelegate` |

### 2. Ключевые принципы современного delegate-паттерна (2026)

- **weak ссылка** на delegate — обязательно, чтобы избежать retain cycle
- **AnyObject** — протокол обычно ограничивается `AnyObject` (только классы)
- **optional методы** — через `@objc optional` (старый UIKit-стиль) или default-реализация в протоколе (новый стиль)
- **Односторонняя связь** — делегатор знает о делегате, но делегат не знает о делегаторе
- **Множество делегатов** — редко, но возможно через массив weak-ссылок

### 3. Самый современный и рекомендуемый паттерн 2026 года

```swift
// 1. Протокол с ограничением AnyObject (только классы)
protocol DownloadDelegate: AnyObject {
    func downloadDidStart()
    func downloadDidProgress(_ progress: Double)
    func downloadDidFinish(with data: Data)
    func downloadDidFail(with error: Error)
}

// 2. Класс-делегатор
final class FileDownloader {
    weak var delegate: DownloadDelegate?
    
    private var task: URLSessionDataTask?
    
    func startDownload(from url: URL) {
        delegate?.downloadDidStart()
        
        task = URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
            guard let self else { return }
            
            if let error {
                self.delegate?.downloadDidFail(with: error)
                return
            }
            
            guard let data else {
                self.delegate?.downloadDidFail(with: NSError(domain: "No data", code: -1))
                return
            }
            
            self.delegate?.downloadDidFinish(with: data)
        }
        
        task?.resume()
    }
    
    func cancel() {
        task?.cancel()
    }
}

// 3. Класс-делегат (часто UIViewController или ViewModel)
final class DownloadViewController: UIViewController, DownloadDelegate {
    
    private let downloader = FileDownloader()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        downloader.delegate = self
        
        let url = URL(string: "https://example.com/file.pdf")!
        downloader.startDownload(from: url)
    }
    
    func downloadDidStart() {
        showLoadingIndicator()
    }
    
    func downloadDidProgress(_ progress: Double) {
        updateProgressBar(progress)
    }
    
    func downloadDidFinish(with data: Data) {
        hideLoadingIndicator()
        saveFile(data)
    }
    
    func downloadDidFail(with error: Error) {
        hideLoadingIndicator()
        showErrorAlert(error.localizedDescription)
    }
    
    deinit {
        downloader.cancel()  // важно отменить задачу
    }
}
```

### 4. Современные альтернативы delegate в 2026 году

| Ситуация                                      | Старый стиль (delegate)                              | Новый стиль (2026)                                   | Когда выбрать новый |
|-----------------------------------------------|-------------------------------------------------------|------------------------------------------------------|---------------------|
| UITableView / UICollectionView                | `dataSource` / `delegate`                             | `UICollectionViewDiffableDataSource` + `UICollectionViewCellRegistration` | Почти всегда (iOS 13+) |
| UITextField / UITextView                      | `UITextFieldDelegate`                                 | Combine: `publisher(for: \.text)` или SwiftUI `onChange` | SwiftUI / Combine-проекты |
| Асинхронные операции                          | Completion handler                                    | `async throws` + `await`                             | Новый код (iOS 15+) |
| Уведомления о состоянии                       | Delegate с несколькими методами                       | `Publisher` / `AsyncStream` / `Observation`          | Combine / SwiftUI |
| Передача данных между экранами                | Delegate + метод `didSelect`                          | `@Environment`, `@Observable`, `ObservableObject`    | SwiftUI             |

### 5. Лучшие практики delegate в Swift 2026

- **Всегда** делай протокол с ограничением `AnyObject` → `weak var delegate`  
- **Не делай слишком много методов** в протоколе — лучше несколько узких протоколов  
- **Используй optional методы через default-реализацию** (новый стиль):

```swift
protocol DownloadDelegate: AnyObject {
    func downloadDidStart()
    func downloadDidFinish()
}

extension DownloadDelegate {
    func downloadDidStart() {}  // default — пустая реализация
}
```

- **Добавляй deinit с отменой** — если делегатор держит задачи/таймеры  
- **Swift 6 strict concurrency** — delegate должен быть `Sendable` или помечен `@MainActor`  
- **Документируйте** — пиши комментарий «weak var delegate — уведомления о прогрессе загрузки»

**Короткий девиз 2026**:
> Delegate — это когда объект говорит: «если что-то случится — позвони мне, я решу».  
> В 2026 году:  
> - `weak var delegate: AnyObject?` — всегда  
> - несколько узких протоколов лучше одного жирного  
> - новый код → `async/await`, Combine, SwiftUI вместо delegate  
> - старый UIKit → delegate всё ещё король

Удачи с чистыми и безопасными делегатами в твоём коде! 📞