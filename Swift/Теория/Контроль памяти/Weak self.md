#memory #arc #weak-self #retain-cycle #closures #swift #memory-management

---
### Определение
**weak self** — это механизм захвата self в замыканиях ([[closure]]s) с использованием слабой ссылки, которая не увеличивает счетчик ссылок объекта ([[retain count]]) . Это предотвращает создание цикла сильных ссылок ([[retain cycle]]), когда объект удерживает замыкание, а замыкание — объект .

В [[Swift]] ссылки по умолчанию являются **сильными ([[strong]])**. Использование `[weak self]` в capture list замыкания преобразует [[self]] в опциональную ([[Optional]]) слабую ссылку, которая автоматически становится [[nil]], когда объект освобождается из памяти .

### Зачем это знать iOS-разработчику?
1.  **Предотвращение утечек памяти:** Главная причина использования — избежать retain cycles .
2.  **Асинхронные операции:** Сетевые запросы, таймеры, анимации — все, где замыкание может жить дольше, чем объект .
3.  **Безопасность:** Слабая ссылка автоматически обнуляется, предотвращая обращение к освобожденной памяти .
4.  **Делегаты:** В паттерне делегата также используется `weak` для предотвращения циклов .
5.  **Производительность:** Правильное управление памятью делает приложение более стабильным .

---

### Основные термины

| Термин | Описание |
|--------|----------|
| **Strong Reference** | Сильная ссылка, удерживает объект в памяти (счетчик +1) |
| **Weak Reference** | Слабая ссылка, не удерживает объект (счетчик не меняется) |
| **Retain Cycle** | Ситуация, когда два или более объекта удерживают друг друга → утечка памяти |
| **Closure Capture List** | Место в замыкании (`[weak self, weak delegate]`), где определяются правила захвата |
| **Optional Self** | `self` внутри `[weak self]` становится опциональным (`self?`) |

---

### Базовый синтаксис

```swift
class MyClass {
    var value = 10
    
    func doSomething() {
        // Без weak self — потенциальный retain cycle
        DispatchQueue.main.async {
            print(self.value)  // self захвачен сильно
        }
        
        // С weak self — безопасно
        DispatchQueue.main.async { [weak self] in
            print(self?.value ?? 0)  // self опциональный
        }
        
        // С weak self и guard
        DispatchQueue.main.async { [weak self] in
            guard let self = self else { return }
            print(self.value)  // self неопциональный внутри guard
        }
    }
}
```

---

### Примеры от простого к сложному

#### Уровень 1: Базовый weak self

```swift
class ViewController: UIViewController {
    var name = "Main Screen"
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Сильный захват — утечка!
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            print(self.name)  // self захвачен сильно
        }
        
        // Слабый захват — безопасно
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            print(self?.name ?? "No name")
        }
    }
}
```

#### Уровень 2: weak self с [[guard let]]

```swift
class DataLoader {
    var data: [String] = []
    
    func loadData() {
        APIService.shared.fetchData { [weak self] result in
            guard let self = self else { return }
            
            switch result {
            case .success(let newData):
                self.data = newData
                self.updateUI()
            case .failure(let error):
                self.showError(error)
            }
        }
    }
    
    func updateUI() { }
    func showError(_ error: Error) { }
}
```

#### Уровень 3: Множественный захват

```swift
class ProfileViewController: UIViewController {
    var user: User?
    weak var delegate: ProfileDelegate?
    
    func loadProfile() {
        UserService.shared.fetchProfile { [weak self, weak delegate] result in
            guard let self = self else { return }
            
            switch result {
            case .success(let user):
                self.user = user
                self.updateUI()
                delegate?.profileDidLoad(user)
            case .failure(let error):
                self.showError(error)
            }
        }
    }
}
```

#### Уровень 4: Таймеры и weak self

```swift
class TimerExample {
    var count = 0
    var timer: Timer?
    
    func startTimer() {
        // Опасно! Таймер удерживает замыкание, замыкание удерживает self
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            guard let self = self else { return }
            self.count += 1
            print(self.count)
        }
    }
    
    func stopTimer() {
        timer?.invalidate()
        timer = nil
    }
    
    deinit {
        stopTimer()
        print("TimerExample deinit")
    }
}

var example: TimerExample? = TimerExample()
example?.startTimer()
DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    example = nil  // объект освободится благодаря weak self
}
```

#### Уровень 5: Замыкания как свойства класса

```swift
class NetworkManager {
    var onComplete: ((Result<Data, Error>) -> Void)?
    
    func fetchData() {
        // Опасно! onComplete удерживается классом
        onComplete = { [weak self] result in
            guard let self = self else { return }
            self.handleResult(result)
        }
        
        // Симуляция запроса
        DispatchQueue.global().async {
            let result = Result<Data, Error>.success(Data())
            DispatchQueue.main.async {
                self.onComplete?(result)
            }
        }
    }
    
    func handleResult(_ result: Result<Data, Error>) {
        // обработка
    }
}
```

#### Уровень 6: Анимации и [[UIView]].animate

```swift
class AnimationViewController: UIViewController {
    @IBOutlet weak var boxView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // В UIView.animate weak self не нужен, так как замыкание не удерживается
        UIView.animate(withDuration: 1.0) {
            self.boxView.alpha = 0  // безопасно
        }
        
        // Но если внутри замыкания есть дополнительное замыкание
        UIView.animate(withDuration: 1.0, animations: {
            self.boxView.alpha = 0
        }) { [weak self] _ in
            self?.navigationController?.popViewController(animated: true)
        }
    }
}
```

#### Уровень 7: Вложенные замыкания

```swift
class NestedClosuresExample {
    var data: [Int] = []
    
    func process() {
        someAsyncOperation { [weak self] in
            guard let self = self else { return }
            
            self.anotherAsyncOperation { [weak self] in
                guard let self = self else { return }
                self.data.append(1)
            }
        }
    }
    
    func someAsyncOperation(completion: @escaping () -> Void) {
        DispatchQueue.global().async { completion() }
    }
    
    func anotherAsyncOperation(completion: @escaping () -> Void) {
        DispatchQueue.global().async { completion() }
    }
}
```

#### Уровень 8: RxSwift/Combine и weak self

```swift
import Combine

class CombineViewModel: ObservableObject {
    @Published var data: [String] = []
    var cancellables = Set<AnyCancellable>()
    
    func setupBindings() {
        // В Combine обычно используют weak self в sink
        NotificationCenter.default.publisher(for: .dataUpdated)
            .sink { [weak self] _ in
                self?.loadData()
            }
            .store(in: &cancellables)
    }
    
    func loadData() {
        // загрузка данных
    }
}
```

---

### weak vs unowned

| Характеристика         | weak                                             | unowned                                        |
| ---------------------- | ------------------------------------------------ | ---------------------------------------------- |
| **Становится nil**     | Да                                               | Нет                                            |
| **Optional**           | Всегда optional                                  | Может быть [[non-optional]]                    |
| **Безопасность**       | Высокая (проверка через [[if let]])              | Низкая (crash при обращении после [[dealloc]]) |
| **Производительность** | Чуть медленнее                                   | Быстрее                                        |
| **Когда использовать** | По умолчанию, когда объект может быть освобожден | Когда объект гарантированно живет дольше       |

```swift
class Child {
    weak var parent: Parent?  // parent может быть nil
    unowned let owner: Owner   // owner должен жить дольше
}
```

**Правило:** Используйте `weak` по умолчанию. `unowned` — только когда 100% уверены в жизненном цикле .

---

### Распространенные ошибки

#### 1. **Забытый weak self**

```swift
// Ошибка! Retain cycle!
class ViewController {
    var callback: (() -> Void)?
    
    func setup() {
        callback = {
            self.doSomething()  // self захвачен сильно
        }
    }
}
```

#### 2. **Неправильное использование unowned**

```swift
// Опасно! Может упасть
class ViewController {
    func setup() {
        someAsyncCall { [unowned self] in
            self.doSomething()  // crash, если self уже освобожден
        }
    }
}
```

#### 3. **Забытая опциональность**

```swift
// Ошибка! self? нужно развернуть
someAsyncCall { [weak self] in
    print(self.name)  // Value of optional type 'ViewController?' must be unwrapped
}
```

#### 4. **Weak self в замыканиях, которые не удерживаются**

```swift
// Избыточно, но не вредно
UIView.animate(withDuration: 0.3) { [weak self] in
    self?.view.alpha = 0  // weak self здесь не нужен, но и не навредит
}
```

---

### Лучшие практики

#### 1. **Всегда используйте weak self в замыканиях, которые сохраняются**

```swift
class ViewModel {
    var onUpdate: (() -> Void)?
    
    func setup() {
        onUpdate = { [weak self] in
            self?.updateUI()
        }
    }
}
```

#### 2. **Используйте guard let self = self для удобства**

```swift
someAsyncCall { [weak self] in
    guard let self = self else { return }
    self.doSomething()  // self неопциональный
    self.updateUI()
}
```

#### 3. **Для множественных захватов**

```swift
someAsyncCall { [weak self, weak delegate] in
    guard let self = self else { return }
    delegate?.didFinish()
    self.process()
}
```

#### 4. **В Combine используйте weak self в sink**

```swift
publisher
    .sink { [weak self] value in
        self?.handle(value)
    }
    .store(in: &cancellables)
```

#### 5. **Проверяйте [[deinit]] для отладки**

```swift
class ViewController {
    deinit {
        print("\(self) deinitialized")  // должен вызываться при закрытии
    }
}
```

#### 6. **Используйте Memory Graph Debugger для поиска утечек**

1.  Запустите приложение
2.  Нажмите на кнопку Memory Graph в панели Debug
3.  Ищите красные циклы

---

### Итог

**weak self** — это essential-инструмент для предотвращения утечек памяти в Swift:

1.  Используется в **замыканиях, которые сохраняются** (свойства класса, асинхронные вызовы, таймеры)
2.  `self` становится **[[Optional]]**, поэтому требует разворачивания (`?` или `guard let`)
3.  **Никогда не создает retain cycle**
4.  Предпочтительнее `unowned` для большинства случаев
5.  Обязателен в:
    -   Замыканиях, присвоенных свойствам класса
    -   Таймерах с замыканиями
    -   Длительных асинхронных операциях
    -   Наблюдателях ([[Combine]], [[NotificationCenter]])

**Золотое правило:** Если ваше замыкание может жить дольше, чем объект, который его захватывает — используйте `[weak self]` .