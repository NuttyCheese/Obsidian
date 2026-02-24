- **[[Publisher]]** — источник данных (например, изменения текста в [[UITextField]], события от [[URLSession]], таймеры)
    
- **[[Subscriber]]** — получатель данных, который обрабатывает значения и завершение
    
- **[[Operators]]** — методы для преобразования, фильтрации, комбинирования потоков
    
- **[[Subscription]]** — связь между Publisher и Subscriber, позволяет управлять жизненным циклом подписки (отписаться и т.п.)
    

---

## 2. Установка

Combine встроен в iOS 13+, подключать ничего не надо, просто импортируйте:

```swift
import Combine
```

---

## 3. Пример 1. Отслеживаем изменения [[UITextField]] через Combine

В [[UIKit]] для текстовых полей Combine не даёт готового `Publisher`, но можно использовать [[NotificationCenter]] или расширения.

### Пример с `UITextField` и `NotificationCenter`

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private var textField = UITextField()
    private var label = UILabel()
    private var cancellables = Set<AnyCancellable>() // для хранения подписок

    override func viewDidLoad() {
        super.viewDidLoad()

        setupViews()

        // Подписываемся на уведомления об изменениях текста
        NotificationCenter.default.publisher(for: UITextField.textDidChangeNotification, object: textField)
            .compactMap { ($0.object as? UITextField)?.text } // извлекаем текст
            .sink { [weak self] text in
                self?.label.text = "Вы ввели: \(text)"
            }
            .store(in: &cancellables)
    }

    private func setupViews() {
        textField.borderStyle = .roundedRect
        textField.placeholder = "Введите текст"

        label.textColor = .black

        view.addSubview(textField)
        view.addSubview(label)

        textField.translatesAutoresizingMaskIntoConstraints = false
        label.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            textField.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            textField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            textField.widthAnchor.constraint(equalToConstant: 200),

            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.topAnchor.constraint(equalTo: textField.bottomAnchor, constant: 20)
        ])
    }
}
```

---

## 4. Пример 2. Использование `@Published` и подписка из ViewModel в UIKit

Часто Combine применяется для организации [[MVVM (Model-View-ViewModel) Architecture|MVVM]]-паттерна. В модели представления (ViewModel) используем `@Published` свойства, в контроллере — подписываемся на них.

```swift
import UIKit
import Combine

// ViewModel с @Published
class MyViewModel {
    @Published var username: String = ""
    @Published var isValid: Bool = false

    private var cancellables = Set<AnyCancellable>()

    init() {
        // Пример простой валидации username (минимум 3 символа)
        $username
            .map { $0.count >= 3 }
            .assign(to: \.isValid, on: self)
            .store(in: &cancellables)
    }
}

// ViewController, подписываемся на изменения ViewModel
class MyViewController: UIViewController {

    private var viewModel = MyViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let usernameTextField = UITextField()
    private let validationLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()

        // Подписываемся на изменения username
        usernameTextField.addTarget(self, action: #selector(textChanged), for: .editingChanged)

        // Подписываемся на isValid для обновления UI
        viewModel.$isValid
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isValid in
                self?.validationLabel.text = isValid ? "Имя валидно" : "Имя слишком короткое"
                self?.validationLabel.textColor = isValid ? .green : .red
            }
            .store(in: &cancellables)
    }

    @objc private func textChanged() {
        viewModel.username = usernameTextField.text ?? ""
    }

    private func setupViews() {
        usernameTextField.borderStyle = .roundedRect
        usernameTextField.placeholder = "Введите имя пользователя"

        validationLabel.textColor = .red

        view.addSubview(usernameTextField)
        view.addSubview(validationLabel)

        usernameTextField.translatesAutoresizingMaskIntoConstraints = false
        validationLabel.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            usernameTextField.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            usernameTextField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            usernameTextField.widthAnchor.constraint(equalToConstant: 200),

            validationLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            validationLabel.topAnchor.constraint(equalTo: usernameTextField.bottomAnchor, constant: 10)
        ])
    }
}
```

---

## 5. Пример 3. Использование Combine с сетевыми запросами

Combine отлично подходит для работы с [[URLSession]].

```swift
import UIKit
import Combine

struct Post: Codable {
    let id: Int
    let title: String
}

class NetworkService {

    func fetchPosts() -> AnyPublisher<[Post], Error> {
        let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!

        return URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: [Post].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main) // обновления UI на главном потоке
            .eraseToAnyPublisher()
    }
}

class PostsViewController: UIViewController {

    private var cancellables = Set<AnyCancellable>()
    private let networkService = NetworkService()

    override func viewDidLoad() {
        super.viewDidLoad()

        networkService.fetchPosts()
            .sink(receiveCompletion: { completion in
                switch completion {
                case .finished:
                    print("Загрузка завершена")
                case .failure(let error):
                    print("Ошибка: \(error)")
                }
            }, receiveValue: { posts in
                print("Посты: \(posts)")
            })
            .store(in: &cancellables)
    }
}
```

---

## 6. Управление жизненным циклом подписок

Очень важно **сохранять подписки** в свойстве типа `Set<AnyCancellable>` в классе, чтобы подписка жила, пока жив объект (например, [[UIViewController]]), и не утекала память.

---

## 7. Краткое резюме

- Импортируем Combine
    
- Подписываемся на Publishers ([[NotificationCenter]], `@Published`, [[URLSession]]`.dataTaskPublisher`)
    
- Используем операторы ([[map]], [[filter]], [[compactMap]], debounce и т.п.)
    
- Сохраняем подписки в `Set<AnyCancellable>`
    
- Обновляем UI на главном потоке через `.receive(on: DispatchQueue.main)`
    
- Можно удобно строить [[MVVM (Model-View-ViewModel) Architecture|MVVM]], где ViewModel — источник данных через `@Published`, а ViewController подписывается
    

---
