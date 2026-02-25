**UIStepper** — это элемент управления в UIKit, который позволяет пользователю **увеличивать или уменьшать значение** с помощью двух кнопок («+» и «–»).

Это классический контроллер для ввода числовых значений с шагом (step), часто используется для настройки количества, громкости, возраста, количества товаров и т.д.

### Основные свойства UIStepper (актуально на 2026 год)

| Свойство                          | Тип / Значение по умолчанию                          | Что делает / зачем нужно                                      | Самый частый сценарий |
|-----------------------------------|-------------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `value`                           | `Double` (0.0)                                        | Текущее значение                                              | Основное свойство, к которому привязываются данные |
| `minimumValue`                    | `Double` (0.0)                                        | Минимально допустимое значение                                | Ограничение снизу |
| `maximumValue`                    | `Double` (100.0)                                      | Максимально допустимое значение                               | Ограничение сверху |
| `stepValue`                       | `Double` (1.0)                                        | Шаг изменения при каждом нажатии                              | Настройка чувствительности |
| `continuous`                      | `Bool` (true)                                         | Отправлять событие `.valueChanged` при каждом шаге (true) или только после отпускания (false) | `false` для экономии вызовов |
| `autorepeat`                      | `Bool` (true)                                         | Повторять изменение при долгом удержании кнопки               | Удобство ввода больших значений |
| `isContinuous` (устаревшее)       | `Bool`                                                | То же, что `continuous` (deprecated в iOS 13+)                | Не используйте |
| `isEnabled`                       | `Bool` (true)                                         | Включён / выключен                                            | Отключение при недоступности |
| `tintColor`                       | `UIColor`                                             | Цвет кнопок и разделителя                                     | Кастомизация под бренд |

### Самый популярный и рекомендуемый паттерн 2026 года (UIKit + MVVM)

```swift
import UIKit

class QuantityViewController: UIViewController {
    
    private let viewModel = QuantityViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var stepper: UIStepper = {
        let s = UIStepper()
        s.minimumValue = 1
        s.maximumValue = 99
        s.stepValue = 1
        s.value = Double(viewModel.quantity)
        s.autorepeat = true
        s.isContinuous = true
        s.addTarget(self, action: #selector(stepperValueChanged), for: .valueChanged)
        s.translatesAutoresizingMaskIntoConstraints = false
        return s
    }()
    
    private lazy var quantityLabel: UILabel = {
        let lbl = UILabel()
        lbl.textAlignment = .center
        lbl.font = .systemFont(ofSize: 32, weight: .bold)
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        let stack = UIStackView(arrangedSubviews: [quantityLabel, stepper])
        stack.axis = .horizontal
        stack.spacing = 20
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    private func bindViewModel() {
        // Двусторонняя привязка через Combine
        viewModel.$quantity
            .map { String($0) }
            .assign(to: \.text, on: quantityLabel)
            .store(in: &cancellables)
        
        viewModel.$quantity
            .map { Double($0) }
            .assign(to: \.value, on: stepper)
            .store(in: &cancellables)
    }
    
    @objc private func stepperValueChanged(_ sender: UIStepper) {
        viewModel.quantity = Int(sender.value)
    }
}

// ViewModel
@MainActor
class QuantityViewModel: ObservableObject {
    @Published var quantity: Int = 1
}
```

### Современные альтернативы UIStepper в 2026 году

| Компонент / Способ                        | Когда лучше UIStepper                              | Минимальная версия | Плюсы |
|-------------------------------------------|-----------------------------------------------------|---------------------|-------|
| **Stepper** в SwiftUI                     | Новый проект или SwiftUI-экран                     | iOS 13+             | Декларативный стиль, проще привязка |
| **UIStepper + Combine**                   | UIKit + MVVM                                       | iOS 13+             | Реактивность, двусторонняя привязка |
| **UITextField + +/- кнопки**              | Требуется точный ввод или большие значения         | iOS 13+             | Гибкость, клавиатура |
| **UISlider**                              | Когда нужен непрерывный диапазон                   | iOS 13+             | Лучше для громкости, яркости |
| **UISegmentedControl**                    | Маленькое фиксированное количество значений        | iOS 13+             | Быстрый выбор |

### Лучшие практики UIStepper в 2026 году

- **Всегда** задавайте `minimumValue` и `maximumValue` — иначе пользователь может задать некорректное значение  
- **Используйте** `stepValue` не только 1 — удобно для 0.5, 10, 100 и т.д.  
- **Для больших диапазонов** — добавляйте рядом `UITextField` с синхронизацией  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue`  
- **Для SwiftUI** — используйте нативный `Stepper` — UIStepper нужен только в UIKit или смешанных проектах  
- **Для Combine** — привязывайте `value` через `.publisher(for: \.value)` или `@Published` в ViewModel  
- **Документируйте** — пишите комментарий:

```swift
/// UIStepper для выбора количества товаров (1–99)
private lazy var quantityStepper: UIStepper = {
    let s = UIStepper()
    s.minimumValue = 1
    s.maximumValue = 99
    s.stepValue = 1
    s.value = Double(viewModel.quantity)
    return s
}()
```

**Короткий итог 2026**:
> `UIStepper` — классический контроллер для **пошагового изменения числового значения** («+» / «–»).  
> В 2026 году:  
> - ключевые свойства — `value`, `minimumValue`, `maximumValue`, `stepValue`, `continuous`  
> - идеален для количества товаров, возраста, громкости, настроек  
> - в SwiftUI — используйте нативный `Stepper`  
> - в UIKit — синхронизируйте с `@Published` через Combine или target-action  
> - это **надёжный** и **доступный** элемент интерфейса, который до сих пор активно используется  

Удачи с удобными и интуитивными элементами ввода в твоём приложении! ➕➖