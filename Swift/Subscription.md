Когда ты создаешь **[[Publisher]]** (например, для отслеживания изменений в переменной или действия пользователя), ты можешь подписаться на этот Publisher с помощью **[[Subscriber]]**. Когда происходят изменения или события (например, текст в текстовом поле меняется), Publisher отправляет эти изменения подписчику через Subscription.

### Как это работает?

1. **Publisher** генерирует данные.
2. **Subscriber** подписывается на Publisher.
3. **Subscription** контролирует этот процесс, управляя потоком данных от Publisher к Subscriber.
4. Когда подписчик больше не интересуется данными, он может отписаться, и поток данных прекращается.

### Основные типы Subscription

1. **[[Cancellable]]**: Это основной тип подписки в [[Combine]]. Когда ты создаешь подписку, она возвращает объект `Cancellable`, который можно использовать для отмены подписки.
    
2. **[[AnyCancellable]]**: Это обертка вокруг `Cancellable`, которая автоматически отменяет подписку, когда она деинициализируется.
    

### Простой пример

Приведем пример с текстом в [[UITextField]] и покажем, как подписаться на изменения с помощью Combine и правильно управлять подписками.

#### Пример 1: Подписка на изменения в `UITextField`

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    // Текстовое поле
    private var textField: UITextField!
    
    // Храним подписки
    private var cancellables = Set<AnyCancellable>()

    override func viewDidLoad() {
        super.viewDidLoad()

        // Инициализируем UITextField
        textField = UITextField()
        textField.borderStyle = .roundedRect
        textField.frame = CGRect(x: 50, y: 100, width: 300, height: 40)
        view.addSubview(textField)
        
        // Подписка на изменения текста в UITextField
        textField
            .publisher(for: \.text) // Создаем Publisher для текстового поля
            .compactMap { $0 } // Убираем nil значения
            .sink { newText in
                print("Текст в UITextField изменился на: \(newText)")
            }
            .store(in: &cancellables) // Сохраняем подписку в коллекцию cancellables
    }
}
```

### Что происходит в этом примере?

1. **Publisher для `UITextField`**: Мы используем `publisher(for: \.text)` для создания Publisher, который отслеживает изменения текста в [[UITextField]]. Когда текст изменяется, Publisher отправляет новое значение в Subscriber.
    
2. **Подписка через [[sink]]**: Мы подписываемся на изменения текста с помощью метода `sink`. Этот метод выполняет замыкание каждый раз, когда текст изменяется.
    
3. **Подписка сохраняется**: Мы сохраняем подписку с помощью `store(in: &cancellables)`. Это позволяет нам отписаться от потока, когда `ViewController` уничтожается (например, когда экран закрывается).
    
4. **[[Cancellable]]**: Это коллекция, в которой мы храним все активные подписки. Важно помнить, что если подписки не сохранены, они могут быть автоматически уничтожены до того, как данные будут переданы.
    

### Пример 2: Отмена подписки

Иногда бывает нужно отменить подписку, чтобы не продолжать получать данные. Для этого используется метод `cancel` или просто удержание подписки в `cancellables`, которая автоматически отписывается, когда объект уничтожается.

#### Добавим отмену подписки:

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    // Текстовое поле
    private var textField: UITextField!
    
    // Храним подписки
    private var cancellable: AnyCancellable?

    override func viewDidLoad() {
        super.viewDidLoad()

        // Инициализируем UITextField
        textField = UITextField()
        textField.borderStyle = .roundedRect
        textField.frame = CGRect(x: 50, y: 100, width: 300, height: 40)
        view.addSubview(textField)
        
        // Подписка на изменения текста в UITextField
        cancellable = textField
            .publisher(for: \.text)
            .compactMap { $0 }
            .sink { newText in
                print("Текст изменился: \(newText)")
            }
        
        // Для демонстрации отменим подписку через 5 секунд
        DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
            self.cancellable?.cancel()
            print("Подписка отменена")
        }
    }
}
```

### Что изменилось в этом примере?

1. Мы сохраняем подписку в свойстве `cancellable`, которое может быть использовано для отмены подписки в дальнейшем.
    
2. После 5 секунд мы отменяем подписку с помощью `cancellable?.cancel()`. Это останавливает поток данных, и больше никаких изменений в текстовом поле не будут отслеживаться.
    
3. Подписка отменяется вручную, и больше не будет вызываться замыкание.
    

### Пример 3: Использование Subscription для работы с массивами

Предположим, у нас есть массив строк, и мы хотим отслеживать его изменения.

```swift
import Combine

class ViewController: UIViewController {
    
    @Published var items: [String] = []
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Подписываемся на изменения в массиве items
        $items
            .sink { updatedItems in
                print("Массив обновлен: \(updatedItems)")
            }
            .store(in: &cancellables)

        // Изменяем массив
        items.append("New Item")
    }
}
```

### Что происходит в этом примере?

1. **Publisher для массива**: Мы создаем Publisher для свойства `items`, которое имеет аннотацию `@Published`. Это позволяет отслеживать изменения массива.
    
2. **Подписка через `sink`**: Мы подписываемся на [[Publisher]] с помощью `sink`, чтобы отслеживать изменения в массиве и выводить его на консоль.
    
3. **Изменение данных**: Когда мы добавляем новый элемент в массив с помощью `items.append("New Item")`, подписка срабатывает, и мы выводим обновленный массив.
    

### Пример 4: Комбинирование нескольких подписок

Иногда бывает полезно подписываться на несколько Publishers одновременно и объединять их.

```swift
import Combine

class ViewController: UIViewController {
    
    @Published var firstName: String = ""
    @Published var lastName: String = ""
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Подписываемся на изменения двух свойств
        Publishers.CombineLatest($firstName, $lastName)
            .sink { firstName, lastName in
                print("Полное имя: \(firstName) \(lastName)")
            }
            .store(in: &cancellables)
        
        // Изменяем значения
        firstName = "John"
        lastName = "Doe"
    }
}
```

### Что происходит в этом примере?

1. **[[CombineLatest]]**: Мы используем `CombineLatest`, чтобы комбинировать два [[Publisher]] (для `firstName` и `lastName`), и подписываемся на их изменения.
    
2. **Получаем комбинированные значения**: Когда одно из значений изменяется, подписка срабатывает, и мы выводим полное имя.
    
3. **Обновление значений**: Мы изменяем значения `firstName` и `lastName`, и подписка выводит обновленное полное имя.
    

### Заключение

**Subscription** в [[Combine]] позволяет нам контролировать потоки данных между **[[Publisher]]** и **[[Subscriber]]**. С помощью подписок можно:

- Получать данные.
- Отслеживать изменения.
- Отменять подписки, когда они больше не нужны.

**`AnyCancellable`** и **`Cancellable`** — это основные механизмы для управления жизненным циклом подписки, и важно правильно хранить подписки, чтобы не создавать утечек памяти или нежеланных повторных подписок.

---