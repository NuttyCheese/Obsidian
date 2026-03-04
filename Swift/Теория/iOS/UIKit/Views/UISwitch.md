#uiswitch #uikit #toggle #settings #ios #swift #controls #accessibility #darkmode #valuechanged

---
**(переключатель / тумблер / свитч)**

**UISwitch** — это классический элемент управления в [[UIKit]], который позволяет пользователю выбрать **одно из двух взаимоисключающих состояний**:

- **ON** (включено, true)
- **OFF** (выключено, false)

Это самый популярный и интуитивный способ реализации бинарных настроек в [[iOS]]-приложениях 2026 года: включить/выключить Wi-Fi, тёмную тему, уведомления, автосохранение, доступ к камере и т.д.

### 1. Когда использовать UISwitch в 2026 году

| Сценарий                                      | Почему именно UISwitch (а не UIButton / UISegmentedControl) | Альтернатива в SwiftUI |
|-----------------------------------------------|---------------------------------------------------------------|-------------------------|
| Включение/выключение функции в настройках     | Интуитивный жест (свайп), мгновенная обратная связь           | `Toggle`                |
| Тёмная/светлая тема                           | Стандартный элемент для системных настроек                    | `Toggle(isOn:)` + `.preferredColorScheme` |
| Разрешения (камера, микрофон, геолокация)     | Пользователь сразу понимает: вкл/выкл                         | `Toggle`                |
| Автоматический режим / ручной режим           | Два чётких состояния                                          | `Toggle`                |
| Фильтры / режимы (ночь/день, звук/без звука)  | Быстрое переключение без лишних подтверждений                 | `Toggle`                |

### 2. Основные свойства UISwitch (2026 актуально)

| Свойство                   | Тип / Значение по умолчанию   | Что делает / зачем нужно                             | Самый частый сценарий        |
| -------------------------- | ----------------------------- | ---------------------------------------------------- | ---------------------------- |
| `isOn`                     | [[Bool]] (false)              | Текущее состояние (true = ON, false = OFF)           | Чтение/запись состояния      |
| `onTintColor`              | [[UIColor]]`?` (system green) | Цвет фона в состоянии ON                             | Брендовый цвет               |
| `thumbTintColor`           | `UIColor?` (white)            | Цвет кружка-ползунка                                 | Контраст с onTintColor       |
| `tintColor`                | `UIColor?` (system gray)      | Цвет рамки/обводки в состоянии OFF                   | Редко меняют                 |
| `isEnabled`                | `Bool` (true)                 | Включён/выключен (серый, не реагирует на касания)    | Отключение при недоступности |
| `isOn` (setter)            | —                             | `mySwitch.setOn(true, animated: true)` — с анимацией | Программное переключение     |
| `addTarget(_:action:for:)` | —                             | Привязка действия к `.valueChanged`                  | Реакция на изменение         |

### 3. Самый популярный паттерн 2026 года  
(UISwitch + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]] + динамическое обновление UI)

```swift
import UIKit
import Combine

class SettingsViewController: UIViewController {
    
    private let viewModel = SettingsViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var darkModeSwitch: UISwitch = {
        let s = UISwitch()
        s.isOn = viewModel.isDarkModeEnabled
        s.onTintColor = .systemBlue
        s.thumbTintColor = .white
        s.addTarget(self, action: #selector(switchChanged), for: .valueChanged)
        s.translatesAutoresizingMaskIntoConstraints = false
        return s
    }()
    
    private lazy var statusLabel: UILabel = {
        let lbl = UILabel()
        lbl.text = viewModel.isDarkModeEnabled ? "Тёмная тема: ВКЛ" : "Тёмная тема: ВЫКЛ"
        lbl.font = .systemFont(ofSize: 17)
        lbl.textColor = .label
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Настройки"
        view.backgroundColor = .systemBackground
        
        let stack = UIStackView(arrangedSubviews: [statusLabel, darkModeSwitch])
        stack.axis = .horizontal
        stack.spacing = 16
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Switch → ViewModel
        darkModeSwitch.publisher(for: \.isOn)
            .removeDuplicates()
            .assign(to: \.isDarkModeEnabled, on: viewModel)
            .store(in: &cancellables)
        
        // ViewModel → UI
        viewModel.$isDarkModeEnabled
            .map { $0 ? "Тёмная тема: ВКЛ" : "Тёмная тема: ВЫКЛ" }
            .assign(to: \.text, on: statusLabel)
            .store(in: &cancellables)
        
        viewModel.$isDarkModeEnabled
            .map { $0 ? UIColor.systemBlue : UIColor.systemGray }
            .assign(to: \.onTintColor, on: darkModeSwitch)
            .store(in: &cancellables)
    }
    
    @objc private func switchChanged(_ sender: UISwitch) {
        // Лёгкая вибрация для обратной связи
        let generator = UIImpactFeedbackGenerator(style: .light)
        generator.impactOccurred()
    }
}

// ViewModel
@MainActor
class SettingsViewModel: ObservableObject {
    @Published var isDarkModeEnabled: Bool = UserDefaults.standard.bool(forKey: "darkMode")
    
    init() {
        $isDarkModeEnabled
            .sink { enabled in
                UserDefaults.standard.set(enabled, forKey: "darkMode")
                // Применяем тему (в реальном проекте — через traitCollectionDidChange)
            }
            .store(in: &cancellables)
    }
}
```

### 4. Лучшие практики UISwitch в 2026 году

- **isOn** — используйте `setOn(_:animated:)` для программного изменения с анимацией  
- **Цвета** — `onTintColor` для ON-состояния, `thumbTintColor` для кружка, `tintColor` для OFF-рамки  
- **Анимация** — добавляйте `UIView.animate` в обработчике `.valueChanged` для плавного изменения UI  
- **Combine** — `publisher(for: \.isOn)` — самый чистый способ двусторонней привязки  
- **Доступность** — `accessibilityLabel = "Тёмная тема"` и `accessibilityValue = isOn ? "Включено" : "Выключено"`  
- **Для SwiftUI** — используйте `Toggle` — UISwitch нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
/// UISwitch для включения тёмной темы с динамической обратной связью
private lazy var darkModeSwitch: UISwitch = {
    let s = UISwitch()
    s.isOn = viewModel.isDarkModeEnabled
    s.onTintColor = .systemBlue
    return s
}()
```

**Короткий итог 2026**:
> **UISwitch** — **бинарный переключатель** для двух состояний (ON/OFF).  
> В 2026 году:  
> - ключевые свойства — `isOn`, `onTintColor`, `thumbTintColor`  
> - самый популярный паттерн — `UISwitch` + Combine + MVVM + вибрация  
> - идеален для настроек, разрешений, режимов, фильтров  
> - в SwiftUI — заменяется на `Toggle`  
> - это **надёжный** и **интуитивный** элемент, который до сих пор используется почти в каждом приложении  
