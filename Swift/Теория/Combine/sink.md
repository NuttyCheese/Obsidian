**`sink`** — это самый простой и часто используемый способ **подписаться** на **Publisher** в Combine.  

Он позволяет указать, что делать, когда придёт:

- новое значение (`receiveValue`)
- завершение потока (`receiveCompletion`) — успешно или с ошибкой

`s sink` возвращает объект **`AnyCancellable`**, который нужно сохранить, чтобы подписка не отменилась преждевременно.

### Основные формы метода `sink` (актуально на 2026 год)

```swift
// Вариант 1: только значения (completion игнорируется)
publisher.sink { value in
    // обработка каждого нового значения
}

// Вариант 2: полный контроль (самый рекомендуемый)
publisher.sink(
    receiveCompletion: { completion in
        switch completion {
        case .finished:
            print("Поток успешно завершён")
        case .failure(let error):
            print("Ошибка:", error)
        }
    },
    receiveValue: { value in
        print("Получено значение:", value)
    }
)
```

### Самые популярные и рекомендуемые паттерны использования sink (2026)

#### 1. Подписка на @Published в ViewModel (MVVM + UIKit)

```swift
class ProfileViewModel: ObservableObject {
    @Published var name = "Аноним"
    @Published var isLoading = false
}

class ProfileViewController: UIViewController {
    private let viewModel = ProfileViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Подписка на изменения имени
        viewModel.$name
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newName in
                self?.nameLabel.text = newName
            }
            .store(in: &cancellables)
        
        // Подписка на состояние загрузки
        viewModel.$isLoading
            .sink { [weak self] isLoading in
                if isLoading {
                    self?.activityIndicator.startAnimating()
                } else {
                    self?.activityIndicator.stopAnimating()
                }
            }
            .store(in: &cancellables)
    }
}
```

#### 2. Обработка сетевого запроса (URLSession + sink)

```swift
URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .decode(type: [User].self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink { completion in
        switch completion {
        case .finished:
            print("Загрузка завершена успешно")
        case .failure(let error):
            print("Ошибка загрузки:", error.localizedDescription)
            // показать алерт
        }
    } receiveValue: { [weak self] users in
        self?.users = users
        self?.tableView.reloadData()
    }
    .store(in: &cancellables)
```

#### 3. Подписка на кастомный PassthroughSubject (события)

```swift
let buttonTap = PassthroughSubject<Void, Never>()

buttonTap
    .sink { [weak self] _ in
        print("Кнопка нажата!")
        self?.performAction()
    }
    .store(in: &cancellables)

// где-то в UI
@objc func buttonPressed() {
    buttonTap.send(())
}
```

#### 4. Обработка ошибок + завершения (самый безопасный вариант)

```swift
networkService.fetchData()
    .receive(on: DispatchQueue.main)
    .sink(
        receiveCompletion: { [weak self] completion in
            self?.isLoading = false
            
            switch completion {
            case .finished:
                print("Успех")
            case .failure(let error):
                self?.showError(error)
            }
        },
        receiveValue: { [weak self] data in
            self?.updateUI(with: data)
        }
    )
    .store(in: &cancellables)
```

### Лучшие практики использования sink в Combine 2026

- **Всегда** используйте полную форму `sink(receiveCompletion:..., receiveValue:...)` — это позволяет корректно обрабатывать ошибки  
- **Никогда** не забывайте `.store(in: &cancellables)` — иначе подписка будет жить вечно → утечка памяти  
- **Для UI-обновлений** — обязательно `.receive(on: DispatchQueue.main)` перед `.sink`  
- **Используйте `[weak self]`** в замыканиях внутри `sink` — это предотвращает retain cycle  
- **Для простых случаев** — можно использовать только `receiveValue`, но это менее безопасно  
- **Для тестирования** — используйте `XCTest` + `XCTestExpectation` внутри `sink`  
- **Документируйте** — всегда пишите комментарий:

```swift
// Обновляем UI при изменении имени пользователя
viewModel.$name
    .receive(on: DispatchQueue.main)
    .sink { [weak self] newName in
        self?.nameLabel.text = newName
    }
    .store(in: &cancellables)
```

**Короткий итог 2026**:
> `sink` — это **основной способ подписаться** на Publisher в Combine.  
> В 2026 году:  
> - используйте полную форму с `receiveCompletion` и `receiveValue`  
> - всегда сохраняйте подписку в `Set<AnyCancellable>`  
> - для UI — `.receive(on: .main)` + `[weak self]`  
> - это **самый частый** и **самый понятный** способ получать данные из Combine-потока  

Удачи с чистыми и безопасными подписками в твоём проекте! 📡