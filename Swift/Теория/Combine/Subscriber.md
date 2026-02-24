**Subscriber** — это **получатель** событий от **Publisher** в Combine.

Он определяет, **что делать**, когда приходят:
- новые значения (receiveValue)
- завершение потока (receiveCompletion) — успешно или с ошибкой

Subscriber — это **активная** часть паттерна: именно он **запрашивает** данные у Publisher (через demand) и **реагирует** на них.

### Основные способы создания Subscriber в Combine (2026 актуально)

| Способ создания Subscriber          | Когда использовать                                      | Плюсы / Минусы | Пример |
|-------------------------------------|----------------------------------------------------------|----------------|--------|
| `.sink(receiveCompletion:..., receiveValue:...)` | Самый универсальный и популярный способ                  | Полный контроль над value и completion | `.sink { print($0) }` |
| `.assign(to:on:)`                   | Прямая привязка значения к свойству объекта              | Очень лаконично, но только для успешных значений | `.assign(to: \.text, on: label)` |
| `.assign(to:)` (iOS 14+)            | Привязка к `@Published` свойству                         | Самый чистый синтаксис для MVVM | `.assign(to: &$viewModel.isValid)` |
| `Subscribers.Sink` (ручной)         | Когда нужно кастомное поведение подписчика               | Полный контроль, редко нужен | `Subscribers.Sink(receiveValue: ...)` |
| `Subscribers.Assign` (ручной)       | Редко — обычно используют удобные методы                 | —              | — |

### Самый рекомендуемый паттерн 2026 года — полный sink

```swift
publisher
    .receive(on: DispatchQueue.main)           // для UI-обновлений
    .sink(
        receiveCompletion: { [weak self] completion in
            switch completion {
            case .finished:
                print("Поток завершён успешно")
                self?.hideLoadingIndicator()
            case .failure(let error):
                print("Ошибка:", error)
                self?.showError(error)
            }
        },
        receiveValue: { [weak self] value in
            self?.updateUI(with: value)
        }
    )
    .store(in: &cancellables)
```

### Топ-10 самых частых сценариев использования sink в 2026

1. Обновление UI из `@Published` свойства ViewModel  
2. Обработка сетевого ответа (`URLSession.dataTaskPublisher`)  
3. Реакция на пользовательский ввод (UITextField.textPublisher)  
4. Обработка кастомных событий (PassthroughSubject.send)  
5. Отслеживание состояния загрузки / ошибки  
6. Валидация формы в реальном времени  
7. Подписка на уведомления NotificationCenter  
8. Реакция на изменения в Core Data / Realm / GRDB  
9. Обработка результатов асинхронных операций (Future, async/await → Future)  
10. Логирование / аналитика событий

### Полный реальный пример (UIKit + MVVM + sink)

```swift
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
            .delay(for: .seconds(1.5), scheduler: DispatchQueue.main)
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
        viewModel.$userName
            .receive(on: DispatchQueue.main)
            .assign(to: \.text, on: nameLabel)
            .store(in: &cancellables)
        
        viewModel.$isLoading
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isLoading in
                if isLoading {
                    self?.loadingIndicator.startAnimating()
                } else {
                    self?.loadingIndicator.stopAnimating()
                }
            }
            .store(in: &cancellables)
    }
}
```

### Лучшие практики sink в Combine 2026

- **Всегда** используйте полную форму с `receiveCompletion` — это позволяет корректно обрабатывать ошибки  
- **Никогда** не забывайте `.store(in: &cancellables)` — иначе подписка живёт вечно → утечка  
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
> `sink` — это **основной способ подписаться** на Publisher и обработать значения / ошибки / завершение.  
> В 2026 году:  
> - используйте полную форму: `sink(receiveCompletion:..., receiveValue:...)`  
> - всегда сохраняйте подписку в `Set<AnyCancellable>`  
> - для UI — `.receive(on: .main)` + `[weak self]`  
> - это **самый понятный** и **самый часто используемый** способ получить данные из Combine  

Удачи с чистыми и надёжными подписками в твоём проекте! 📡