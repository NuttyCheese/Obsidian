**UIPickerView** — это классический элемент управления в [[UIKit]], который позволяет пользователю **выбирать одно значение** из набора предопределённых вариантов, прокручивая «барабан» (колесо, как в старых слот-машинах или спиннерах).

В 2026 году UIPickerView **по-прежнему активно используется** в UIKit-приложениях, хотя в новых проектах на [[SwiftUI]] чаще применяют `Picker`.

### Основные характеристики UIPickerView

| Свойство / Метод                     | Тип / Значение по умолчанию   | Что делает / зачем нужно                                 | Самый частый сценарий         |
| ------------------------------------ | ----------------------------- | -------------------------------------------------------- | ----------------------------- |
| `dataSource`                         | [[UIPickerViewDataSource]]`?` | Источник данных (кол-во компонентов, строк в компоненте) | Обязательно задавать          |
| `delegate`                           | [[UIPickerViewDelegate]]`?`   | Отвечает за отображение строк, выбор, ширину компонентов | Обязательно задавать          |
| `numberOfComponents`                 | [[Int]]                       | Количество «колёс» (обычно 1–3)                          | Дата/время, адрес             |
| `selectedRow(inComponent:)`          | `Int`                         | Текущая выбранная строка в компоненте                    | Чтение/установка выбора       |
| `selectRow(_:inComponent:animated:)` | `Void`                        | Программно выбрать строку с анимацией или без            | Установка начального значения |
| `showsSelectionIndicator`            | [[Bool]] (true)               | Показывать ли синюю полоску выбора (устарело в iOS 13+)  | `.false` — современный стиль  |
| `backgroundColor` / `tintColor`      | [[UIColor]]`?`                | Цвет фона и акцента                                      | Кастомизация                  |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UIPickerView + [[UIKit]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]] + [[Combine]])

```swift
import UIKit
import Combine

class SettingsViewController: UIViewController {
    
    private let viewModel = SettingsViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var pickerView: UIPickerView = {
        let picker = UIPickerView()
        picker.dataSource = self
        picker.delegate = self
        picker.backgroundColor = .systemBackground
        picker.layer.borderColor = UIColor.systemGray4.cgColor
        picker.layer.borderWidth = 1
        picker.layer.cornerRadius = 10
        picker.translatesAutoresizingMaskIntoConstraints = false
        return picker
    }()
    
    private lazy var selectedLabel: UILabel = {
        let lbl = UILabel()
        lbl.textAlignment = .center
        lbl.font = .systemFont(ofSize: 18, weight: .medium)
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Настройки"
        view.backgroundColor = .systemBackground
        
        view.addSubview(pickerView)
        view.addSubview(selectedLabel)
        
        NSLayoutConstraint.activate([
            pickerView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            pickerView.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -40),
            pickerView.widthAnchor.constraint(equalToConstant: 280),
            pickerView.heightAnchor.constraint(equalToConstant: 200),
            
            selectedLabel.topAnchor.constraint(equalTo: pickerView.bottomAnchor, constant: 20),
            selectedLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        bindViewModel()
        
        // Начальный выбор
        pickerView.selectRow(viewModel.selectedIndex, inComponent: 0, animated: false)
        updateSelectedLabel()
    }
    
    private func bindViewModel() {
        viewModel.$selectedOption
            .receive(on: DispatchQueue.main)
            .sink { [weak self] option in
                self?.selectedLabel.text = "Выбрано: \(option)"
            }
            .store(in: &cancellables)
    }
    
    private func updateSelectedLabel() {
        let selectedIndex = pickerView.selectedRow(inComponent: 0)
        viewModel.selectedIndex = selectedIndex
    }
}

// MARK: - UIPickerViewDataSource & UIPickerViewDelegate
extension SettingsViewController: UIPickerViewDataSource, UIPickerViewDelegate {
    
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        1
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        viewModel.options.count
    }
    
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        viewModel.options[row]
    }
    
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        viewModel.selectedIndex = row
    }
    
    func pickerView(_ pickerView: UIPickerView, rowHeightForComponent component: Int) -> CGFloat {
        44  // Удобная высота строки
    }
}

// ViewModel
@MainActor
class SettingsViewModel: ObservableObject {
    let options = ["Лёгкий", "Средний", "Сильный", "Максимальный"]
    
    @Published var selectedIndex: Int = 1 {
        didSet {
            selectedOption = options[selectedIndex]
        }
    }
    
    @Published var selectedOption: String = ""
    
    init() {
        selectedOption = options[selectedIndex]
    }
}
```

### Современные альтернативы UIStepper в 2026 году

| Компонент / Способ                        | Когда лучше UIPickerView                           | Минимальная версия | Плюсы |
|-------------------------------------------|-----------------------------------------------------|---------------------|-------|
| **Picker** в SwiftUI                      | Новый проект или SwiftUI-экран                     | iOS 13+             | Декларативный стиль, `.pickerStyle(.wheel)` / `.menu` |
| **UIPickerView + Combine**                | UIKit + MVVM                                       | iOS 9+              | Реактивность, кастомизация |
| **UISegmentedControl**                    | Маленькое фиксированное количество вариантов (2–5)  | iOS 13+             | Быстрее и проще |
| **UITextField + UIPickerView как inputView** | Когда нужен ввод с клавиатурой + выбор из списка   | iOS 13+             | Гибридный ввод |
| **Menu / UIMenu** (iOS 14+)               | Когда вариантов много или нужны подменю             | iOS 14+             | Более современный вид |

### Лучшие практики UIPickerView в 2026 году

- **Всегда** реализуйте `dataSource` и `delegate` — без них ничего не отобразится  
- **Задавайте** `rowHeightForComponent` — стандарт 44 pt выглядит лучше всего  
- **Для большого количества строк** — используйте `titleForRow` вместо `viewForRow` (проще и быстрее)  
- **Для кастомного вида** — реализуйте `viewForRow` с `UILabel` или `UIView`  
- **Для Combine** — используйте `.publisher(for: \.selectedRow(inComponent:))` или `@Published` в ViewModel  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityValue` для каждой строки  
- **Для SwiftUI** — используйте `Picker` — UIPickerView нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UIPickerView для выбора уровня сложности (Лёгкий / Средний / Сильный)
private lazy var difficultyPicker: UIPickerView = {
    let picker = UIPickerView()
    picker.dataSource = self
    picker.delegate = self
    return picker
}()
```

**Короткий итог 2026**:
> `UIPickerView` — классический «барабан» для выбора одного значения из списка.  
> В 2026 году:  
> - ключевые методы — `dataSource`, `delegate`, `numberOfComponents`, `numberOfRowsInComponent`  
> - идеален для выбора даты, категории, уровня сложности, валюты, единиц измерения  
> - в SwiftUI — заменяется на `Picker`  
> - в UIKit — синхронизируется с `@Published` через Combine или delegate-методы  
> - это **надёжный** и **доступный** элемент, который до сих пор активно используется  
