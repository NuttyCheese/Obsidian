**UIColorPickerViewControllerDelegate** — это протокол в [[UIKit]], который определяет методы-делегаты для обработки событий от **[[UIColorPickerViewController]]** (контроллер выбора цвета, появившийся в iOS 14).

Он нужен, если вы хотите:
- получать выбранный цвет в реальном времени (по мере движения пальца),
- реагировать на подтверждение выбора (Done),
- обрабатывать отмену.

### Обязательные и ключевые методы (2025–2026 актуально)

| Метод делегата                                      | Когда вызывается                                                                 | Что обычно делают внутри метода                                                                 | Обязателен? |
|-----------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|-------------|
| `colorPickerViewControllerDidSelectColor(_:)`       | Пользователь выбрал цвет (вызывается **многократно** при движении пальца)       | Обновление UI в реальном времени (preview цвета, live-изменение фона, текста и т.д.)            | Нет (но самый частый) |
| `colorPickerViewControllerDidFinish(_:)`            | Пользователь нажал **Done** (подтвердил выбор)                                  | Сохранение окончательного цвета, закрытие контроллера, применение изменения                     | Нет |
| `colorPickerViewControllerDidCancel(_:)`            | Пользователь нажал **Cancel** или смахнул вниз                                  | Закрытие контроллера без сохранения, откат изменений (если нужно)                                | Нет |

**Важно**:  
Все три метода **не обязательны**, но без `colorPickerViewControllerDidSelectColor` контроллер бесполезен — вы просто не получите выбранный цвет.

### Полный рекомендуемый шаблон 2026 года

```swift
import UIKit

class ColorPickerExampleViewController: UIViewController, UIColorPickerViewControllerDelegate {
    
    private let colorPreviewView = UIView()
    private var selectedColor: UIColor = .systemBlue
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        // Preview выбранного цвета
        colorPreviewView.backgroundColor = selectedColor
        colorPreviewView.layer.cornerRadius = 16
        colorPreviewView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(colorPreviewView)
        
        NSLayoutConstraint.activate([
            colorPreviewView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            colorPreviewView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            colorPreviewView.widthAnchor.constraint(equalToConstant: 200),
            colorPreviewView.heightAnchor.constraint(equalToConstant: 200)
        ])
        
        // Кнопка открытия пикера
        let button = UIButton(type: .system)
        button.setTitle("Выбрать цвет", for: .normal)
        button.titleLabel?.font = .preferredFont(forTextStyle: .headline)
        button.addTarget(self, action: #selector(openColorPicker), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            button.topAnchor.constraint(equalTo: colorPreviewView.bottomAnchor, constant: 40),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    @objc private func openColorPicker() {
        let picker = UIColorPickerViewController()
        picker.supportsAlpha = true                     // поддержка прозрачности
        picker.selectedColor = selectedColor            // начальный цвет
        picker.delegate = self
        
        // Современные настройки (iOS 14+)
        picker.title = "Выберите цвет"
        picker.preferredContentSize = CGSize(width: 400, height: 500)
        
        present(picker, animated: true)
    }
    
    // Самый важный метод — live-обновление при движении пальца
    func colorPickerViewControllerDidSelectColor(_ viewController: UIColorPickerViewController) {
        selectedColor = viewController.selectedColor
        colorPreviewView.backgroundColor = selectedColor
        
        // Можно сразу применять в реальном времени
        // view.backgroundColor = selectedColor.withAlphaComponent(0.2)
    }
    
    // Пользователь нажал Done
    func colorPickerViewControllerDidFinish(_ viewController: UIColorPickerViewController) {
        viewController.dismiss(animated: true)
        
        // Сохраняем окончательный выбор
        print("Выбран окончательный цвет:", selectedColor)
        
        // Применяем глобально или сохраняем в UserDefaults
        // UserDefaults.standard.setColor(selectedColor, forKey: "userAccentColor")
    }
    
    // Пользователь отменил (нажал Cancel или смахнул вниз)
    func colorPickerViewControllerDidCancel(_ viewController: UIColorPickerViewController) {
        viewController.dismiss(animated: true)
        print("Выбор цвета отменён")
        
        // Можно откатить preview к предыдущему значению
        // colorPreviewView.backgroundColor = previousColor
    }
}
```

### Полезные настройки UIColorPickerViewController (2026)

```swift
let picker = UIColorPickerViewController()

picker.supportsAlpha = true              // разрешить прозрачность (по умолчанию true)
picker.selectedColor = .systemPurple     // начальный цвет
picker.title = "Выберите акцентный цвет" // заголовок (iPad)
picker.delegate = self

// Дополнительные опции (редко используются)
picker.preferredContentSize = CGSize(width: 420, height: 600) // размер на iPad
```

### Лучшие практики UIColorPickerViewControllerDelegate в 2026

- **Всегда** реализуйте `colorPickerViewControllerDidSelectColor` — это основной способ получать live-цвет  
- **Используйте** `didFinish` для финального сохранения и закрытия  
- **В `didCancel`** — просто закрывайте и (опционально) откатывайте preview  
- **Поддерживайте** прозрачность (`supportsAlpha = true`) — это стандарт 2025–2026  
- **Для [[SwiftUI]]** — используйте [[UIColorPicker]] (нативный компонент с iOS 14) — он проще и не требует делегата  
- **Для сохранения** — используйте [[UIColor]] + [[UserDefaults]] или [[Codable]] структуру с `CodableColor`  
- **Документируйте** — пишите комментарий «UIColorPickerViewControllerDelegate — live-обновление цвета в реальном времени + сохранение при Done»

**Короткий итог 2026**:
> `UIColorPickerViewControllerDelegate` — это делегат для обработки выбора цвета из системного пикера.  
> В 2026 году:  
> - главный метод — `colorPickerViewControllerDidSelectColor` (live-обновление)  
> - финальное сохранение — в `colorPickerViewControllerDidFinish`  
> - отмена — в `colorPickerViewControllerDidCancel`  
> - это **единственный** способ получить выбранный цвет из нативного UIColorPickerViewController в UIKit  
