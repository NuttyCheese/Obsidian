#uipickerviewdatasource #uikit #picker #datasource #ios #swift #selection #inputview #accessibility #dynamic-type

---
**(источник данных для UIPickerView / поставщик данных пикера)**

**UIPickerViewDataSource** — это **обязательный** протокол в UIKit, который отвечает за **количество** и **структуру** данных в **[[UIPickerView]]** (классический «барабан» / колесо выбора).

Он работает **в паре** с [[UIPickerViewDelegate]] (который отвечает за отображение строк и реакцию на выбор).

Без реализации этого протокола **UIPickerView не покажет ни одной строки** и будет пустым.

### Основные методы протокола (все обязательные)

| Метод делегата                           | Что возвращает | Когда вызывается                         | Самый частый сценарий                    |
| ---------------------------------------- | -------------- | ---------------------------------------- | ---------------------------------------- |
| `numberOfComponents(in:)`                | [[Int]]        | Сколько «колёс» (компонентов) показывать | Обычно 1, иногда 2–3 (дата/время, адрес) |
| `pickerView(_:numberOfRowsInComponent:)` | `Int`          | Сколько строк в конкретном компоненте    | Количество элементов в списке            |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UIPickerView как inputView для [[UITextField]] + DataSource + Delegate + [[Combine]])

```swift
import UIKit
import Combine

class SettingsViewController: UIViewController, UIPickerViewDataSource, UIPickerViewDelegate {
    
    private let viewModel = SettingsViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var currencyField: UITextField = {
        let tf = UITextField()
        tf.borderStyle = .roundedRect
        tf.textAlignment = .center
        tf.placeholder = "Выберите валюту"
        tf.translatesAutoresizingMaskIntoConstraints = false
        return tf
    }()
    
    private lazy var pickerView: UIPickerView = {
        let picker = UIPickerView()
        picker.dataSource = self
        picker.delegate = self
        picker.backgroundColor = .systemBackground
        return picker
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Настройки"
        view.backgroundColor = .systemBackground
        
        view.addSubview(currencyField)
        NSLayoutConstraint.activate([
            currencyField.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            currencyField.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            currencyField.widthAnchor.constraint(equalToConstant: 280),
            currencyField.heightAnchor.constraint(equalToConstant: 44)
        ])
        
        // Привязываем picker как inputView
        currencyField.inputView = pickerView
        
        // Начальное значение
        currencyField.text = viewModel.currencyOptions[viewModel.selectedCurrencyIndex]
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$selectedCurrencyIndex
            .map { viewModel.currencyOptions[$0] }
            .assign(to: \.text, on: currencyField)
            .store(in: &cancellables)
        
        viewModel.$selectedCurrencyIndex
            .sink { [weak self] index in
                self?.pickerView.selectRow(index, inComponent: 0, animated: true)
            }
            .store(in: &cancellables)
    }
    
    // MARK: - UIPickerViewDataSource (обязательно)
    
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        1  // Один барабан
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        viewModel.currencyOptions.count
    }
    
    // MARK: - UIPickerViewDelegate (рекомендуется)
    
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        viewModel.currencyOptions[row]
    }
    
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        viewModel.selectedCurrencyIndex = row
        
        // Скрываем picker после выбора
        currencyField.resignFirstResponder()
    }
    
    func pickerView(_ pickerView: UIPickerView, rowHeightForComponent component: Int) -> CGFloat {
        44  // Удобная высота строки
    }
}

// ViewModel
@MainActor
class SettingsViewModel: ObservableObject {
    let currencyOptions = ["USD ($)", "EUR (€)", "RUB (₽)", "GBP (£)", "JPY (¥)"]
    
    @Published var selectedCurrencyIndex: Int = 0
}
```

### Лучшие практики UIPickerViewDataSource в 2026 году

- **numberOfComponents** — почти всегда 1 (для простоты), 2–3 — только если действительно нужно (дата/время, рост/вес)
- **numberOfRowsInComponent** — возвращайте реальное количество элементов из модели
- **Кэшируйте** данные — не создавайте массив заново при каждом вызове
- **Синхронизируйте** с моделью через `@Published` + Combine
- **Для динамического контента** — обновляйте `reloadAllComponents()` после изменения данных
- **Для SwiftUI** — используйте `Picker` — UIPickerViewDataSource нужен только в UIKit
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue` для каждой строки через делегат
- **Документируйте** — пишите комментарий:

```swift
extension SettingsViewController: UIPickerViewDataSource {
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        1  // Один столбец выбора валюты
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        viewModel.currencyOptions.count
    }
}
```

**Короткий итог 2026**:
> **UIPickerViewDataSource** — протокол, который **поставляет данные** для UIPickerView (количество колёс и строк).  
> В 2026 году:  
> - ключевые методы — `numberOfComponents`, `numberOfRowsInComponent`  
> - самый популярный сценарий — как inputView для UITextField + синхронизация с моделью  
> - обязателен для любого работающего UIPickerView  
> - в SwiftUI — заменяется на `Picker`  
> - это **фундаментальный** протокол для кастомного выбора из списка в UIKit  
