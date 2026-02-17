**UIColorPickerViewController** — это системный контроллер в [[UIKit]], который позволяет пользователю выбрать цвет с помощью нативного интерфейса [[iOS]] (появился в iOS 14, 2020 год).

Это готовое решение для всех случаев, когда нужно дать пользователю выбрать цвет: темы, заметки, рисование, кастомизация профиля, фильтры фото и т.д.

В 2026 году это **единственный рекомендуемый** способ показать выбор цвета в UIKit-приложениях.

### Основные возможности (актуально на iOS 19 / 2026)

| Свойство / Возможность            | Описание                                                             | Значение по умолчанию / Рекомендация 2026 |
| --------------------------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| **selectedColor**                 | Текущий выбранный цвет ([[UIColor]])                                 | `.systemBlue` или последний выбранный     |
| **supportsAlpha**                 | Разрешить выбор прозрачности (показывать альфа-слайдер)              | `true` — почти всегда                     |
| **title**                         | Заголовок в навигационной панели (опционально)                       | Обычно не нужен, можно оставить пустым    |
| **delegate**                      | [[UIColorPickerViewControllerDelegate]] — уведомления о выборе цвета | Реализуйте для получения результата       |
| **popoverPresentationController** | Настройка popover на iPad / Mac Catalyst                             | Обязательно для iPad                      |
| **presentationStyle**             | `.automatic`, `.pageSheet`, `.formSheet` и т.д.                      | `.automatic` или `.pageSheet`             |

### Самый популярный и современный паттерн 2026 года

#### 1. Простой выбор цвета (самый частый случай)

```swift
class ColorSettingsViewController: UIViewController, UIColorPickerViewControllerDelegate {
    
    private let colorPreview = UIView()
    private var currentColor: UIColor = .systemBlue
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Предпросмотр цвета
        colorPreview.translatesAutoresizingMaskIntoConstraints = false
        colorPreview.backgroundColor = currentColor
        colorPreview.layer.cornerRadius = 12
        view.addSubview(colorPreview)
        
        NSLayoutConstraint.activate([
            colorPreview.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            colorPreview.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            colorPreview.widthAnchor.constraint(equalToConstant: 200),
            colorPreview.heightAnchor.constraint(equalToConstant: 200)
        ])
        
        // Кнопка для открытия пикера
        let pickButton = UIButton(type: .system)
        pickButton.setTitle("Выбрать цвет", for: .normal)
        pickButton.addAction(UIAction { [weak self] _ in
            self?.showColorPicker()
        }, for: .touchUpInside)
        pickButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(pickButton)
        
        NSLayoutConstraint.activate([
            pickButton.topAnchor.constraint(equalTo: colorPreview.bottomAnchor, constant: 40),
            pickButton.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    private func showColorPicker() {
        let picker = UIColorPickerViewController()
        picker.delegate = self
        picker.supportsAlpha = true                     // разрешить прозрачность
        picker.selectedColor = currentColor             // начальный цвет
        
        // Для iPad / Mac — popover
        picker.popoverPresentationController?.sourceView = view
        picker.popoverPresentationController?.sourceRect = view.bounds
        picker.popoverPresentationController?.permittedArrowDirections = .any
        
        present(picker, animated: true)
    }
    
    // Делегат: цвет выбран
    func colorPickerViewControllerDidSelectColor(_ viewController: UIColorPickerViewController) {
        currentColor = viewController.selectedColor
        colorPreview.backgroundColor = currentColor
        
        // Сохраняем выбор (пример)
        UserDefaults.standard.setColor(currentColor, forKey: "userAccentColor")
    }
    
    // Делегат: цвет изменился (пока пользователь тянет слайдер)
    func colorPickerViewControllerDidFinish(_ viewController: UIColorPickerViewController) {
        // Можно обновить UI или сохранить окончательный выбор
    }
}
```

### 2. Расширенный пример: выбор цвета + предпросмотр в реальном времени

```swift
func colorPickerViewControllerDidSelectColor(_ viewController: UIColorPickerViewController) {
    // Обновляем предпросмотр мгновенно
    previewView.backgroundColor = viewController.selectedColor
    
    // Можно сразу применять цвет в приложении
    view.window?.tintColor = viewController.selectedColor
}
```

### 3. Лучшие практики UIColorPickerViewController в Swift 2026

- **Всегда** устанавливайте `supportsAlpha = true` — пользователи ожидают прозрачность  
- **selectedColor** — передавайте текущий цвет, чтобы пользователь видел, что меняет  
- **iPad / Mac Catalyst** — **обязательно** настройте `popoverPresentationController`, иначе краш  
- **Делегат** — реализуйте оба метода:  
  - `didSelectColor` — обновление в реальном времени (пока пользователь тянет слайдер)  
  - `didFinish` — финальное сохранение (когда закрыли пикер)  
- **Не показывайте пикер модально на весь экран** — лучше `.pageSheet` или popover  
- **Сохраняйте выбор** — [[Swift/Теория/Хранение данных/UserDefaults]], [[Core Data]], AppStorage ([[SwiftUI]])  
- **[[@MainActor]]** — все обновления UI — на главном акторе  
- **Swift 6 strict concurrency** — UIColorPickerViewController полностью безопасен  
- **Документируйте** — пиши комментарий «UIColorPickerViewController — выбор акцентного цвета с предпросмотром в реальном времени»

**Короткий девиз 2026**:
> UIColorPickerViewController — это **нативный системный пикер цвета** от Apple: красивый, современный, поддерживает альфа, автоматически адаптируется под тему и доступность.  
> В 2026 году это **единственный правильный** способ дать пользователю выбрать цвет в UIKit.  
> Всегда указывай `supportsAlpha`, начальный `selectedColor` и настраивай popover на iPad.
