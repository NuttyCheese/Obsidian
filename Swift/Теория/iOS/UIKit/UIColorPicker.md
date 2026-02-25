**UIColorPicker** (точнее — **[[UIColorPickerViewController]]**) — это системный контроллер в UIKit, представленный в **iOS 14** (2020), который позволяет пользователю **выбирать цвет** с помощью интуитивного и современного интерфейса.

Это нативный аналог цветового круга, который мы видим в настройках [[iOS]] (цвет иконок, обоев, акцентов) и во многих приложениях (заметки, рисунки, кастомизация тем).

### Основные возможности UIColorPickerViewController

| Свойство / Метод | Тип / Значение по умолчанию                | Что делает / зачем нужно                       | Самый частый сценарий                          |
| ---------------- | ------------------------------------------ | ---------------------------------------------- | ---------------------------------------------- |
| `selectedColor`  | [[UIColor]]                                | Текущий выбранный цвет (двусторонняя привязка) | Основное свойство                              |
| `supportsAlpha`  | [[Bool]] (true)                            | Разрешить выбор прозрачности (альфа-канал)     | Отключать, если нужен только непрозрачный цвет |
| `title`          | [[String]]`?`                              | Заголовок в navigation bar                     | "Выберите цвет темы"                           |
| `delegate`       | [[UIColorPickerViewControllerDelegate]]`?` | Уведомления о выборе цвета и отмене            | Реакция на выбор                               |
| `init()`         | —                                          | Простое создание                               | —                                              |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UIColorPickerViewController]] + [[Combine]] + [[MVVM (Model-View-ViewModel) Architecture|MVVM]])

```swift
import UIKit
import Combine

class ThemeSettingsViewController: UIViewController {
    
    private let viewModel = ThemeViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var colorPreview: UIView = {
        let v = UIView()
        v.layer.cornerRadius = 12
        v.layer.borderWidth = 1
        v.layer.borderColor = UIColor.systemGray4.cgColor
        v.translatesAutoresizingMaskIntoConstraints = false
        return v
    }()
    
    private lazy var pickColorButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Выбрать цвет темы", for: .normal)
        btn.titleLabel?.font = .systemFont(ofSize: 17, weight: .medium)
        btn.addTarget(self, action: #selector(showColorPicker), for: .touchUpInside)
        btn.translatesAutoresizingMaskIntoConstraints = false
        return btn
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Тема приложения"
        view.backgroundColor = .systemBackground
        
        view.addSubview(colorPreview)
        view.addSubview(pickColorButton)
        
        NSLayoutConstraint.activate([
            colorPreview.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            colorPreview.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -60),
            colorPreview.widthAnchor.constraint(equalToConstant: 180),
            colorPreview.heightAnchor.constraint(equalToConstant: 180),
            
            pickColorButton.topAnchor.constraint(equalTo: colorPreview.bottomAnchor, constant: 40),
            pickColorButton.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Синхронизация цвета из ViewModel → preview
        viewModel.$selectedColor
            .receive(on: DispatchQueue.main)
            .assign(to: \.backgroundColor, on: colorPreview)
            .store(in: &cancellables)
        
        // Обратная связь — если пользователь выбрал цвет где-то ещё
        viewModel.$selectedColor
            .sink { color in
                print("Тема изменена на:", color)
            }
            .store(in: &cancellables)
    }
    
    @objc private func showColorPicker() {
        let picker = UIColorPickerViewController()
        picker.supportsAlpha = false                    // отключаем прозрачность
        picker.title = "Цвет темы"
        picker.selectedColor = viewModel.selectedColor
        picker.delegate = self
        
        present(picker, animated: true)
    }
}

// MARK: - UIColorPickerViewControllerDelegate
extension ThemeSettingsViewController: UIColorPickerViewControllerDelegate {
    
    func colorPickerViewControllerDidFinish(_ viewController: UIColorPickerViewController) {
        // Пользователь нажал "Готово"
        viewModel.selectedColor = viewController.selectedColor
        dismiss(animated: true)
    }
    
    func colorPickerViewControllerDidSelectColor(_ viewController: UIColorPickerViewController) {
        // Пользователь меняет цвет в реальном времени
        viewModel.selectedColor = viewController.selectedColor
    }
}

// ViewModel
@MainActor
class ThemeViewModel: ObservableObject {
    @Published var selectedColor: UIColor = .systemBlue
}
```

### Современные альтернативы UIColorPickerViewController в 2026 году

| Альтернатива                                             | Когда лучше UIColorPickerViewController        | Минимальная версия | Плюсы                               |
| -------------------------------------------------------- | ---------------------------------------------- | ------------------ | ----------------------------------- |
| **ColorPicker** в SwiftUI                                | Новый проект или [[SwiftUI]]-экран             | iOS 14+            | Декларативный стиль, проще привязка |
| **UIColorPickerViewController + Combine**                | UIKit + MVVM                                   | iOS 14+            | Реактивность, двусторонняя привязка |
| **Custom color wheel (SwiftUI / UIKit)**                 | Очень специфический дизайн цветового круга     | iOS 14+            | Полная кастомизация                 |
| **Third-party библиотеки** (Colorful, ChromaColorPicker) | Когда нужен более красивый/анимационный picker | iOS 13+            | Красивые анимации, градиенты        |

### Лучшие практики UIColorPickerViewController в 2026 году

- **Всегда** задавайте `supportsAlpha = false`, если прозрачность не нужна — упрощает UX  
- **Используйте** `delegate` методы `didSelectColor` (реал-тайм) и `didFinish` (подтверждение)  
- **Для Combine** — привязывайте `selectedColor` через `@Published` в ViewModel  
- **Для доступности** — задавайте `accessibilityLabel = "Выбор цвета темы"`  
- **Для SwiftUI** — используйте `ColorPicker` — UIColorPickerViewController нужен только в UIKit  
- **Для предпросмотра** — показывайте выбранный цвет в реальном времени через `didSelectColor`  
- **Документируйте** — пишите комментарий:

```swift
/// UIColorPickerViewController для выбора основного цвета темы приложения
private func showColorPicker() {
    let picker = UIColorPickerViewController()
    picker.supportsAlpha = false
    picker.delegate = self
    present(picker, animated: true)
}
```

**Короткий итог 2026**:
> `UIColorPickerViewController` — системный контроллер для **выбора цвета** с нативным интерфейсом iOS.  
> В 2026 году:  
> - ключевые свойства — `selectedColor`, `supportsAlpha`  
> - ключевые методы делегата — `didSelectColor` (реал-тайм), `didFinish` (подтверждение)  
> - идеален для кастомизации тем, акцентов, цвета текста/фона  
> - в SwiftUI — заменяется на `ColorPicker`  
> - это **стандартный** и **самый надёжный** способ дать пользователю выбрать цвет в UIKit  
