#uipickerviewdelegate #uikit #picker #delegate #ios #swift #selection #inputview #accessibility #dynamic-type

---
**(делегат для UIPickerView / обработчик событий пикера)**

**UIPickerViewDelegate** — это протокол в [[UIKit]], который отвечает за **отображение**, **форматирование** и **реакцию на выбор** строк в **[[UIPickerView]]** (классический "барабан" / колесо выбора).

Он работает **в паре** с [[UIPickerViewDataSource]] (который отвечает только за количество компонентов и строк).

**Важно в 2026 году:**  
UIPickerView всё ещё активно используется в [[UIKit]]-приложениях, особенно для:
- выбора даты/времени (когда [[UIDatePicker]] не подходит по дизайну)
- выбора из списка (валюта, страна, категория, уровень сложности)
- кастомного ввода (часы/минуты, рост/вес, рейтинги)
- как inputView для [[UITextField]]

### Основные методы UIPickerViewDelegate

| Метод делегата                                      | Когда вызывается                                        | Что обычно возвращают / делают   | Самый частый сценарий                    |
| --------------------------------------------------- | ------------------------------------------------------- | -------------------------------- | ---------------------------------------- |
| `pickerView(_:titleForRow:forComponent:)`           | Для каждой видимой строки — нужно вернуть текст         | [[String]]`?` — текст для строки | Самый популярный метод — простой текст   |
| `pickerView(_:attributedTitleForRow:forComponent:)` | То же, но с атрибутами (цвет, шрифт, подчёркивание)     | [[NSAttributedString]]`?`        | Кастомный цвет/шрифт для строк           |
| `pickerView(_:viewForRow:forComponent:reusing:)`    | Нужно вернуть полностью кастомный [[UIView]] для строки | `UIView?`                        | Иконка + текст, градиент, сложный layout |
| `pickerView(_:didSelectRow:inComponent:)`           | Пользователь выбрал строку (после остановки барабана)   | `Void` — реакция на выбор        | Обновление модели, UI, сохранение выбора |
| `pickerView(_:rowHeightForComponent:)`              | Задаёт высоту строки в компоненте                       | [[CGFloat]]                      | Большие строки с иконками                |
| `pickerView(_:widthForComponent:)`                  | Задаёт ширину компонента (колеса)                       | `CGFloat`                        | Разные ширины для разных барабанов       |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UIPickerView как inputView для UITextField + делегат + data source + [[Combine]])

```swift
import UIKit
import Combine

class SettingsViewController: UIViewController, UIPickerViewDelegate, UIPickerViewDataSource {
    
    private let viewModel = SettingsViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var difficultyField: UITextField = {
        let tf = UITextField()
        tf.borderStyle = .roundedRect
        tf.textAlignment = .center
        tf.placeholder = "Выберите сложность"
        tf.translatesAutoresizingMaskIntoConstraints = false
        return tf
    }()
    
    private lazy var pickerView: UIPickerView = {
        let picker = UIPickerView()
        picker.delegate = self
        picker.dataSource = self
        picker.backgroundColor = .systemBackground
        return picker
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Настройки"
        view.backgroundColor = .systemBackground
        
        view.addSubview(difficultyField)
        NSLayoutConstraint.activate([
            difficultyField.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            difficultyField.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            difficultyField.widthAnchor.constraint(equalToConstant: 280),
            difficultyField.heightAnchor.constraint(equalToConstant: 44)
        ])
        
        // 1. Привязываем picker как inputView
        difficultyField.inputView = pickerView
        
        // 2. Начальное значение
        difficultyField.text = viewModel.difficultyOptions[viewModel.selectedDifficultyIndex]
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$selectedDifficultyIndex
            .map { viewModel.difficultyOptions[$0] }
            .assign(to: \.text, on: difficultyField)
            .store(in: &cancellables)
        
        viewModel.$selectedDifficultyIndex
            .sink { [weak self] index in
                self?.pickerView.selectRow(index, inComponent: 0, animated: true)
            }
            .store(in: &cancellables)
    }
    
    // MARK: - UIPickerViewDataSource
    
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        1
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        viewModel.difficultyOptions.count
    }
    
    // MARK: - UIPickerViewDelegate
    
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        viewModel.difficultyOptions[row]
    }
    
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        viewModel.selectedDifficultyIndex = row
        
        // Скрываем клавиатуру после выбора
        difficultyField.resignFirstResponder()
    }
    
    func pickerView(_ pickerView: UIPickerView, rowHeightForComponent component: Int) -> CGFloat {
        44  // Удобная высота строки
    }
}

// ViewModel
@MainActor
class SettingsViewModel: ObservableObject {
    let difficultyOptions = ["Лёгкий", "Средний", "Сложный", "Эксперт"]
    
    @Published var selectedDifficultyIndex: Int = 1
}
```

### Лучшие практики UIPickerViewDelegate в 2026 году

- **titleForRow** — самый простой и быстрый способ (текст)
- **attributedTitleForRow** — когда нужен разный цвет/шрифт для строк
- **viewForRow** — когда нужна иконка, градиент, сложный layout (но медленнее)
- **didSelectRow** — обновляйте модель и скрывайте клавиатуру (`resignFirstResponder()`)
- **rowHeightForComponent** — минимум 44 pt для доступности
- **Для Combine** — синхронизируйте `selectedRow` через `@Published`
- **Для [[SwiftUI]]** — используйте `Picker` — UIPickerViewDelegate нужен только в UIKit
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue` для каждой строки
- **Документируйте** — пишите комментарий:

```swift
extension SettingsViewController: UIPickerViewDelegate {
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        // Обновляем модель и скрываем picker
        viewModel.selectedDifficultyIndex = row
        difficultyField.resignFirstResponder()
    }
}
```

**Короткий итог 2026**:
> **UIPickerViewDelegate** — протокол для **отображения и обработки выбора** в UIPickerView (барабан/колесо).  
> В 2026 году:  
> - ключевые методы — `titleForRow`, `didSelectRow`, `viewForRow` (если нужен кастом)  
> - самый популярный сценарий — как inputView для UITextField + синхронизация с моделью  
> - идеален для выбора из списка (сложность, валюта, страна, единицы измерения)  
> - в SwiftUI — заменяется на `Picker`  
> - это **надёжный** и **до сих пор активно используемый** способ кастомного ввода в UIKit  
