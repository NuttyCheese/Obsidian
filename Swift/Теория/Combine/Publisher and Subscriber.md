**Publisher / Subscriber** — это фундаментальный паттерн реактивного программирования, лежащий в основе фреймворка **Combine** (и многих других библиотек, таких как [[RxSwift]], ReactiveSwift и т.д.).

В Combine этот паттерн реализован следующим образом:

- **Publisher** — это **источник** (производитель) значений. Он может:
  - выдавать **0..N значений** (Output)
  - завершиться **успешно** (finished) или **с ошибкой** (Failure)
  - быть **ленивым** — начинает работу только при наличии подписчика

- **Subscriber** — это **потребитель** (получатель) значений. Он:
  - запрашивает у Publisher данные (demand)
  - получает значения, ошибки или завершение
  - может **отменить** подписку в любой момент

Связь между ними — это **Subscription** (подписка), которая управляет жизненным циклом и backpressure (сколько данных можно передать).

### Основные типы Publisher в Combine (2025–2026)

| Тип Publisher                        | Сколько значений | Хранит текущее значение? | Завершается сразу? | Самый частый сценарий в 2026 |
|--------------------------------------|-------------------|---------------------------|---------------------|------------------------------|
| `Just(value)`                        | 1                 | Да                        | Да                  | фиксированное значение, мок, дефолт |
| `Fail(error:)`                       | 0                 | —                         | Да (сразу ошибка)   | явная ошибка в цепочке |
| `Empty(completeImmediately:)`        | 0                 | —                         | Да / Нет            | пустой успешный результат |
| `@Published`                         | 0..N              | Да                        | Нет                 | ViewModel → View в MVVM |
| `PassthroughSubject`                 | 0..N              | Нет                       | Нет                 | события, команды, broadcast |
| `CurrentValueSubject`                | 1..N              | Да                        | Нет                 | состояние с начальным значением |
| `URLSession.dataTaskPublisher`       | 1                 | —                         | Да                  | любой HTTP-запрос |
| `NotificationCenter.default.publisher(for:)` | 0..N          | —                         | Нет                 | UIKit-уведомления |
| `Timer.publish`                      | 0..N              | —                         | Нет                 | периодические обновления |

### Самые популярные паттерны использования Publisher / Subscriber (2026)

#### 1. Простая подписка через `.sink` (самый частый способ)

```swift
import Combine

let publisher = Just("Привет, Combine!")

publisher
    .sink { value in
        print("Значение:", value)
    } receiveCompletion: { completion in
        print("Завершение:", completion)
    }
    .store(in: &cancellables)
```

#### 2. Подписка на `@Published` в ViewModel ([[MVVM (Model-View-ViewModel) Architecture|MVVM]] + [[UIKit]])

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
        
        viewModel.$name
            .receive(on: DispatchQueue.main)
            .assign(to: \.text, on: nameLabel)
            .store(in: &cancellables)
        
        viewModel.$isLoading
            .assign(to: \.isHidden, on: loadingIndicator)
            .store(in: &cancellables)
    }
}
```

#### 3. Сетевой запрос через [[URLSession]]`.dataTaskPublisher`

```swift
struct User: Codable {
    let id: Int
    let name: String
}

func fetchUser(id: Int) -> AnyPublisher<User, Error> {
    let url = URL(string: "https://jsonplaceholder.typicode.com/users/\(id)")!
    
    return URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .decode(type: User.self, decoder: JSONDecoder())
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}

fetchUser(id: 1)
    .sink { completion in
        if case .failure(let error) = completion {
            print("Ошибка:", error)
        }
    } receiveValue: { user in
        print("Пользователь:", user.name)
    }
    .store(in: &cancellables)
```

#### 4. PassthroughSubject как кастомный event bus

```swift
class EventBus {
    static let shared = EventBus()
    
    let userDidLogin = PassthroughSubject<User, Never>()
    let logoutRequested = PassthroughSubject<Void, Never>()
    
    private init() {}
}

// Подписка где угодно в приложении
EventBus.shared.userDidLogin
    .sink { user in
        print("Вход:", user.name)
    }
    .store(in: &globalCancellables)

// Вызов события
EventBus.shared.userDidLogin.send(currentUser)
```

### Лучшие практики работы с Publisher в Combine 2026

- **Всегда** храните подписку: `.store(in: &cancellables)` или `.assign(to:)`  
- **Для UI** — используйте `.receive(on: DispatchQueue.main)` перед обновлением `@Published` или элементов интерфейса  
- **Для ввода** — добавляйте `.debounce`, `.removeDuplicates`, `.filter { !$0.isEmpty }`  
- **Для ошибок** — используйте `.catch`, `.replaceError`, `.retry` — делайте цепочку устойчивой  
- **Для тестов** — создавайте Publisher через `Just`, `Fail`, `PassthroughSubject`, `CurrentValueSubject`  
- **В SwiftUI** — чаще используйте `@Observable` + `.task` — Combine нужен только для сложных цепочек  
- **Документируйте** — пишите комментарий:

```swift
/// Publisher для поиска с отложенной обработкой и фильтрацией пустых запросов
$searchText
    .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
    .removeDuplicates()
    .filter { !$0.isEmpty }
    .flatMap { self.search(query: $0) }
    .assign(to: &$results)
```

**Короткий итог 2026**:
> **Publisher** — это **источник асинхронных данных** в Combine: может выдавать значения, ошибки или завершаться.  
> Самые популярные в 2026:  
> - `@Published` — для ViewModel → View  
> - `URLSession.dataTaskPublisher` — для сети  
> - `PassthroughSubject` — для событий и команд  
> - `Just` — для фиксированных значений  
> - `NotificationCenter.publisher` — для UIKit-уведомлений  
> Подписывайся через `.sink`, `.assign`, храни в `Set<AnyCancellable>`
