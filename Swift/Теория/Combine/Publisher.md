**Publisher** — это **источник данных** в Combine.  
Он **может** выдавать:

- **значения** (Output) — любое количество раз  
- **ошибку** (Failure) — один раз и завершает поток  
- **завершение** (finished) — успешно или с ошибкой

Publisher — это **пассивный** объект: сам по себе он ничего не делает, пока на него **не подпишутся** (Subscriber).  
Как только появляется подписчик — Publisher начинает «толкать» данные.

### Основные типы Publisher (актуальные в 2026 году)

| Тип Publisher                        | Что делает                                                                 | Когда использовать в 2026 | Пример создания |
|--------------------------------------|----------------------------------------------------------------------------|----------------------------|-----------------|
| `Just(value)`                        | Выдаёт **одно значение** и сразу завершается успешно                       | фиксированное значение, мок, дефолт в цепочке | `Just("Hello")` |
| `Fail(error:)`                       | Сразу завершается с ошибкой                                                | явная ошибка в пайплайне   | `Fail(error: URLError(.badURL))` |
| `Empty(completeImmediately:)`        | Ничего не выдаёт, сразу завершается (или не завершается)                   | пустой успешный результат  | `Empty(completeImmediately: true)` |
| `@Published`                         | Хранит текущее значение, выдаёт его новым подписчикам + обновления         | ViewModel → View в MVVM    | `@Published var count = 0` |
| `PassthroughSubject`                 | Не хранит значение, передаёт события всем текущим подписчикам             | события, команды, broadcast | `PassthroughSubject<Void, Never>()` |
| `CurrentValueSubject`                | Хранит текущее значение, сразу выдаёт его новым подписчикам + обновления   | состояние с текущим значением | `CurrentValueSubject<String, Never>("initial")` |
| `URLSession.dataTaskPublisher`       | Асинхронный сетевой запрос                                                 | любой HTTP-запрос          | `URLSession.shared.dataTaskPublisher(for: url)` |
| `NotificationCenter.default.publisher(for:)` | Превращает уведомления в поток данных                                      | UIKit-уведомления          | `NotificationCenter.default.publisher(for: .keyboardWillShow)` |
| `Timer.publish`                      | Периодический таймер                                                       | обновление каждые N секунд | `Timer.publish(every: 1, on: .main, in: .common)` |
| `Future`                             | Одноразовая асинхронная операция (completion-handler → Publisher)          | обёртка над старым API     | `Future { promise in ... }` |

### Как создать и использовать Publisher (самые частые паттерны 2026)

#### 1. Простейший Publisher из массива / последовательности

```swift
let numbers = [1, 2, 3, 4, 5].publisher

numbers
    .sink { value in
        print("Получено:", value)
    }
    .store(in: &cancellables)
// Вывод: 1 2 3 4 5
```

#### 2. @Published в ViewModel (MVVM + Combine + SwiftUI/UIKit)

```swift
class CounterViewModel: ObservableObject {
    @Published var count = 0
    
    func increment() {
        count += 1
    }
}

// В SwiftUI
struct CounterView: View {
    @StateObject var viewModel = CounterViewModel()
    
    var body: some View {
        Text("Счёт: \(viewModel.count)")
            .onReceive(viewModel.$count) { newCount in
                print("Счёт изменился:", newCount)
            }
    }
}

// В UIKit
class CounterViewController: UIViewController {
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewModel.$count
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newCount in
                self?.countLabel.text = "Счёт: \(newCount)"
            }
            .store(in: &cancellables)
    }
}
```

#### 3. Сетевой запрос (URLSession + Combine)

```swift
struct User: Codable {
    let id: Int
    let name: String
}

func fetchUsers() -> AnyPublisher<[User], Error> {
    let url = URL(string: "https://jsonplaceholder.typicode.com/users")!
    
    return URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .decode(type: [User].self, decoder: JSONDecoder())
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}

fetchUsers()
    .sink(receiveCompletion: { completion in
        if case .failure(let error) = completion {
            print("Ошибка:", error)
        }
    }, receiveValue: { users in
        print("Загружено пользователей:", users.count)
    })
    .store(in: &cancellables)
```

#### 4. PassthroughSubject как кастомный event bus

```swift
class EventBus {
    static let shared = EventBus()
    
    let userDidLogin = PassthroughSubject<User, Never>()
    let themeChanged = PassthroughSubject<Theme, Never>()
    
    private init() {}
}

// Подписка где угодно
EventBus.shared.userDidLogin
    .sink { user in
        print("Вход:", user.name)
    }
    .store(in: &globalCancellables)

// Где-то в коде
EventBus.shared.userDidLogin.send(currentUser)
```

### Лучшие практики работы с Publisher в 2026 году

- **Всегда** храните подписку: `.store(in: &cancellables)` или `.assign(to:)`  
- **Для UI** — всегда `.receive(on: DispatchQueue.main)` перед обновлением `@Published` или UI  
- **Для ввода** — добавляйте `.debounce`, `.removeDuplicates`, `.filter` перед тяжёлыми операциями  
- **Для ошибок** — используйте `.catch`, `.replaceError`, `.retry` — делайте пайплайн устойчивым  
- **Для тестов** — используйте `Just`, `Fail`, `PassthroughSubject`, `CurrentValueSubject`  
- **Для SwiftUI** — чаще используйте `@Observable` + `.task` — Combine нужен только в сложных случаях  
- **Документируйте** — пишите комментарий:

```swift
/// Publisher для поиска с отложенной обработкой
$searchText
    .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
    .removeDuplicates()
    .flatMap { self.search(query: $0) }
    .assign(to: &$results)
```

**Короткий итог 2026**:
> Publisher — это **источник асинхронных данных** в Combine: может выдавать значения, ошибки или завершаться.  
> Самые популярные в 2026:  
> - `@Published` — для ViewModel → View  
> - `URLSession.dataTaskPublisher` — для сети  
> - `PassthroughSubject` — для кастомных событий  
> - `Just` — для фиксированных значений  
> - `NotificationCenter.publisher` — для UIKit-уведомлений  
> Подписывайся через `.sink` / `.assign` и храни в `Set<AnyCancellable>`

Удачи с мощными и предсказуемыми потоками данных в твоём проекте! 🔄