**Subscriber** в **Combine** — это объект, который **подписывается** на **Publisher** и **получает** от него события:  

- новые значения (`receiveValue`)  
- завершение потока (`receiveCompletion`) — успешно (`.finished`) или с ошибкой (`.failure`)  

Subscriber — это **активная** сторона: он **запрашивает** данные (demand) и **реагирует** на них.

### Как создаётся Subscriber в реальной практике (2026)

| Способ создания                  | Когда использовать                                      | Плюсы / Минусы | Самый частый пример |
|----------------------------------|----------------------------------------------------------|----------------|---------------------|
| `.sink(receiveCompletion:..., receiveValue:...)` | Универсальный способ — полный контроль над value и completion | Самый популярный и рекомендуемый | `.sink { print($0) }` |
| `.assign(to:on:)`                | Прямая привязка значения к свойству объекта (например UI) | Очень лаконично, но только для успешных значений | `.assign(to: \.text, on: label)` |
| `.assign(to:)` (iOS 14+)         | Привязка к `@Published` свойству другого объекта         | Самый чистый синтаксис для MVVM | `.assign(to: &$viewModel.isValid)` |
| `Subscribers.Sink` (ручной)      | Когда нужен кастомный Subscriber с сложной логикой       | Полный контроль, редко нужен | `Subscribers.Sink(receiveValue: ...)` |
| `Subscribers.Assign` (ручной)    | Редко — обычно используют удобные методы                 | —              | — |

### Самый рекомендуемый паттерн 2026 года — полный sink с обработкой ошибок

```swift
publisher
    .receive(on: DispatchQueue.main)           // ← обязательно для UI
    .sink(
        receiveCompletion: { [weak self] completion in
            switch completion {
            case .finished:
                print("Поток успешно завершён")
                self?.hideLoading()
            case .failure(let error):
                print("Ошибка:", error.localizedDescription)
                self?.showError(error)
            }
        },
        receiveValue: { [weak self] value in
            self?.updateUI(with: value)
        }
    )
    .store(in: &cancellables)
```

### Топ-10 реальных сценариев использования Subscriber в 2026 году

1. Обновление UILabel / UITextField из `@Published` свойства ViewModel  
2. Обработка ответа от `URLSession.dataTaskPublisher`  
3. Реакция на ввод в UITextField (`.publisher(for: \.text)`)  
4. Отслеживание нажатий кнопки через `PassthroughSubject`  
5. Показ/скрытие loading indicator по состоянию `@Published var isLoading`  
6. Валидация формы в реальном времени (email + пароль → isEnabled)  
7. Реакция на системные уведомления (`NotificationCenter.publisher`)  
8. Обновление таблицы/коллекции при изменении массива в ViewModel  
9. Обработка результатов асинхронных операций (`Future`, `async` → `Future`)  
10. Логирование / отправка аналитики при каждом событии

### Полный реальный пример (UIKit + MVVM + Subscriber)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var userName = "Загрузка..."
    @Published var isLoading = true
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        fetchUser()
    }
    
    private func fetchUser() {
        // имитация сети
        Just("Александр Иванов")
            .delay(for: .seconds(1.5), scheduler: RunLoop.main)
            .handleEvents(receiveSubscription: { [weak self] _ in
                self?.isLoading = true
            })
            .sink { [weak self] name in
                self?.userName = name
                self?.isLoading = false
            }
            .store(in: &cancellables)
    }
}

class ProfileViewController: UIViewController {
    
    private let viewModel = ProfileViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var nameLabel: UILabel = {
        let lbl = UILabel()
        lbl.textAlignment = .center
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    private lazy var loadingIndicator: UIActivityIndicatorView = {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.translatesAutoresizingMaskIntoConstraints = false
        return indicator
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        view.addSubview(nameLabel)
        view.addSubview(loadingIndicator)
        
        NSLayoutConstraint.activate([
            nameLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            nameLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            
            loadingIndicator.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            loadingIndicator.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    private func bindViewModel() {
        // Подписчик на имя пользователя
        viewModel.$userName
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newName in
                self?.nameLabel.text = newName
            }
            .store(in: &cancellables)
        
        // Подписчик на состояние загрузки
        viewModel.$isLoading
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isLoading in
                if isLoading {
                    self?.loadingIndicator.startAnimating()
                    self?.nameLabel.isHidden = true
                } else {
                    self?.loadingIndicator.stopAnimating()
                    self?.nameLabel.isHidden = false
                }
            }
            .store(in: &cancellables)
        
        // Подписчик на ошибку (если появится)
        viewModel.$errorMessage
            .receive(on: DispatchQueue.main)
            .sink { [weak self] message in
                if let message {
                    self?.showAlert(message: message)
                }
            }
            .store(in: &cancellables)
    }
    
    private func showAlert(message: String) {
        let alert = UIAlertController(title: "Ошибка", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

### Лучшие практики Subscriber в Combine 2026

- **Всегда** используйте полную форму `sink(receiveCompletion:..., receiveValue:...)` — это позволяет корректно обрабатывать ошибки  
- **Никогда** не забывайте `.store(in: &cancellables)` — иначе подписка живёт вечно → утечка памяти  
- **Для UI** — **обязательно** `.receive(on: DispatchQueue.main)` перед `.sink`  
- **Используйте `[weak self]`** внутри замыканий `sink` — предотвращает retain cycle  
- **Для простых случаев** можно использовать короткую форму `.sink { print($0) }`, но в продакшене — полная  
- **Для отладки** — добавляйте `.handleEvents(receiveSubscription: ..., receiveCancel: ...)`  
- **Документируйте** — всегда пиши комментарий:

```swift
// Обновляем имя пользователя при изменении в ViewModel
viewModel.$userName
    .receive(on: DispatchQueue.main)
    .sink { [weak self] newName in
        self?.nameLabel.text = newName
    }
    .store(in: &cancellables)
```

**Короткий итог 2026**:
> Subscriber — это **получатель** событий от Publisher: получает значения, ошибки и завершение.  
> В 2026 году:  
> - основной способ создания — `.sink(receiveCompletion:..., receiveValue:...)`  
> - всегда сохраняйте подписку в `Set<AnyCancellable>`  
> - для UI — `.receive(on: .main)` + `[weak self]`  
> - это **самый понятный** и **самый часто используемый** способ реагировать на поток данных в Combine  

Удачи с чистыми и надёжными подписками в твоём проекте! 📡