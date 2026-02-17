**Binding** — это механизм, при котором изменения в одном объекте (например, модели данных) автоматически отражаются в другом объекте (например, UI-элементы). В [[Combine]] это достигается через использование **[[Publisher]]** и **[[Subscriber]]**.

### Пример с UITextField

Допустим, у нас есть [[UITextField]], и мы хотим, чтобы его текст синхронизировался с переменной в модели (например, в `ViewModel`). Также изменения этой переменной должны быть отражены в `UITextField`.

#### 1. Создадим ViewModel с @Published

Сначала создадим модель данных, которая будет содержать свойство с текстом, и будем отслеживать его изменения с помощью Combine.

```swift
import UIKit
import Combine

class MyViewModel {
    // Используем @Published, чтобы уведомлять подписчиков об изменениях
    @Published var text: String = ""
}
```

#### 2. Создадим ViewController и UITextField

Теперь создадим ViewController, где будем связывать данные из `UITextField` с моделью. Для этого используем Combine, чтобы слушать изменения текста в поле и синхронизировать их с переменной в `ViewModel`.

```swift
import UIKit
import Combine

class MyViewController: UIViewController {
    
    // Создаем объект ViewModel
    private var viewModel = MyViewModel()
    
    // Создаем UITextField
    private var textField: UITextField!
    
    // Храним подписки для отмены подписок позже
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Настроим UITextField
        textField = UITextField()
        textField.borderStyle = .roundedRect
        textField.frame = CGRect(x: 50, y: 100, width: 300, height: 40)
        view.addSubview(textField)
        
        // Добавляем Publisher для UITextField: когда текст изменяется, подписка на текст
        textField
            .publisher(for: \.text)
            .compactMap { $0 } // Игнорируем nil значения
            .assign(to: &$viewModel.text) // Присваиваем новое значение переменной viewModel.text
        
        // Подписываемся на изменения в ViewModel и обновляем текст в UITextField
        viewModel.$text
            .sink { [weak self] newText in
                self?.textField.text = newText // Обновляем текст в UITextField
            }
            .store(in: &cancellables) // Сохраняем подписку для последующего отмены
    }
}
```

### Что происходит в этом коде?

1. **Publisher для UITextField**: Мы создаем Publisher для `UITextField` с помощью метода `.publisher(for: \.text)`, который отслеживает изменения текста в поле. Когда текст изменяется, он передает новое значение в `viewModel.text`.
    
2. **Синхронизация данных через `@Published`**: В `ViewModel` у нас есть свойство `@Published var text`, которое автоматически уведомляет всех подписчиков (например, наш `ViewController`) о том, что значение изменилось.
    
3. **Подписка на изменения модели**: Мы подписываемся на `viewModel.$text` (Publisher, связанный с `@Published` свойством) и обновляем текст в `UITextField` каждый раз, когда модель изменяет значение.
    
4. **Компактное использование**: Мы используем `compactMap { $0 }`, чтобы игнорировать возможные [[nil]] значения в тексте.
    

### Пример с UISwitch

Теперь давай рассмотрим пример с [[UISwitch]], который мы будем синхронизировать с булевым значением в модели.

#### 1. Модель

```swift
class MySwitchViewModel {
    // Используем @Published для отслеживания состояния переключателя
    @Published var isSwitchOn: Bool = false
}
```

#### 2. ViewController с [[UISwitch]]

```swift
import UIKit
import Combine

class MySwitchViewController: UIViewController {
    
    private var viewModel = MySwitchViewModel()
    private var switchControl: UISwitch!
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Настроим UISwitch
        switchControl = UISwitch()
        switchControl.frame = CGRect(x: 50, y: 150, width: 0, height: 0)
        view.addSubview(switchControl)
        
        // Связываем изменение значения UISwitch с моделью через Combine
        switchControl
            .publisher(for: \.isOn)
            .assign(to: &$viewModel.isSwitchOn) // Присваиваем новое значение переменной viewModel.isSwitchOn
        
        // Подписываемся на изменения в ViewModel
        viewModel.$isSwitchOn
            .sink { [weak self] isOn in
                self?.switchControl.isOn = isOn // Обновляем состояние UISwitch
            }
            .store(in: &cancellables) // Сохраняем подписку
    }
}
```

### Что происходит в этом коде?

1. **Publisher для [[UISwitch]]**: Мы создаем Publisher для `UISwitch` с помощью `.publisher(for: \.isOn)`, который отслеживает изменения состояния переключателя. Когда переключатель изменяет свое состояние, оно передается в `viewModel.isSwitchOn`.
    
2. **Синхронизация через `@Published`**: В `ViewModel` у нас есть свойство `@Published var isSwitchOn`, которое уведомляет всех подписчиков о том, что значение переменной изменилось.
    
3. **Подписка на изменения модели**: Мы подписываемся на `viewModel.$isSwitchOn` и обновляем состояние `UISwitch` каждый раз, когда значение переменной меняется.
    

### Пример с [[UIButton]]

Еще один пример — связывание состояния кнопки с моделью через [[Combine]]. Представим, что мы хотим включать или выключать кнопку в зависимости от состояния некоторой переменной.

#### 1. Модель

```swift
class MyButtonViewModel {
    @Published var isButtonEnabled: Bool = true
}
```

#### 2. ViewController с UIButton

```swift
import UIKit
import Combine

class MyButtonViewController: UIViewController {
    
    private var viewModel = MyButtonViewModel()
    private var button: UIButton!
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Настроим UIButton
        button = UIButton(type: .system)
        button.setTitle("Нажми меня", for: .normal)
        button.frame = CGRect(x: 50, y: 200, width: 200, height: 50)
        view.addSubview(button)
        
        // Связываем состояние кнопки с моделью
        button
            .publisher(for: .touchUpInside)
            .sink { [weak self] _ in
                self?.viewModel.isButtonEnabled.toggle() // Переключаем состояние кнопки
            }
            .store(in: &cancellables)
        
        // Подписываемся на изменения в ViewModel и обновляем состояние кнопки
        viewModel.$isButtonEnabled
            .sink { [weak self] isEnabled in
                self?.button.isEnabled = isEnabled // Включаем/выключаем кнопку
            }
            .store(in: &cancellables)
    }
}
```

### Что происходит в этом коде?

1. **Publisher для UIButton**: Мы создаем Publisher для события `.touchUpInside` кнопки. Когда кнопка нажимается, мы переключаем значение в `viewModel.isButtonEnabled`.
    
2. **Подписка на изменения в модели**: Мы подписываемся на изменения в `viewModel.$isButtonEnabled`, чтобы включать или отключать кнопку в зависимости от значения в модели.
    

### Заключение

**Binding в Combine** — это удобный способ синхронизации данных между моделью и пользовательским интерфейсом. В [[UIKit]] это обычно достигается с использованием Publisher и [[Subscriber]], что позволяет удобно отслеживать изменения и обновлять интерфейс.

В примерах выше мы рассмотрели, как синхронизировать значения между элементами UI (например, `UITextField`, `UISwitch`, `UIButton`) и моделью с помощью [[Combine]]. Это делает код более чистым и позволяет легко отслеживать изменения данных.

---