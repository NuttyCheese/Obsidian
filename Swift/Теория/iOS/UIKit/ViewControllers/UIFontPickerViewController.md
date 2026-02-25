**UIFontPickerViewController** — это системный контроллер Apple для выбора шрифта (появился в [[iOS]] 13, 2019 год).

Он даёт пользователю нативный интерфейс из приложения «Шрифты» (Font Book) — тот же, который используется в Pages, Keynote, Notes и других приложениях Apple.

### Когда стоит использовать UIFontPickerViewController в 2026 году

| Сценарий                                   | Почему именно системный пикер                               | Альтернатива (когда можно обойтись без него) |
|--------------------------------------------|-------------------------------------------------------------|----------------------------------------------|
| Пользователь должен выбрать шрифт вручную | Максимально знакомый интерфейс, поддержка всех установленных шрифтов, фильтры, поиск | Статический список из 5–10 шрифтов |
| Поддержка пользовательских шрифтов (.ttf/.otf) | Автоматически показывает все шрифты, добавленные в «Шрифты» | — |
| iPadOS + Split View / внешняя клавиатура   | Полноценный поиск, категории, preview                       | — |
| Доступность и локализация                  | Всё уже адаптировано Apple (Dynamic Type, RTL, VoiceOver)   | — |
| Минимальный код и современный вид          | Один контроллер вместо кастомного UIPickerView / UITableView | Кастомный список (если нужен очень специфичный UX) |

### Основные свойства и настройка

```swift
let picker = UIFontPickerViewController()

// Самые полезные настройки
picker.delegate = self
picker.title = "Выберите шрифт"
picker.fontPickerConfiguration = UIFontPickerViewController.Configuration(
    filterPredicate: UIFontPickerViewController.Configuration.defaultFilterPredicate, // или кастомный
    includeFaces: true,           // показывать все начертания (Bold, Italic и т.д.)
    displayUsingSystemFont: false // использовать системный шрифт для названий или свой
)

// Наиболее частые фильтры
let onlyMonospaced = UIFontPickerViewController.Configuration(
    filterPredicate: UIFontDescriptor.filterPredicate(for: .monospaced)
)

let onlySerif = UIFontPickerViewController.Configuration(
    filterPredicate: UIFontDescriptor.filterPredicate(for: .serif)
)

// Очень популярный вариант — только шрифты, поддерживающие русский язык
let russianSupported = UIFontPickerViewController.Configuration(
    filterPredicate: UIFontDescriptor.filterPredicate(matchingLanguages: ["ru"])
)

picker.fontPickerConfiguration = russianSupported
```

### Делегат — UIFontPickerViewControllerDelegate

```swift
protocol UIFontPickerViewControllerDelegate: AnyObject {
    func fontPickerViewControllerDidPickFont(_ viewController: UIFontPickerViewController)
    func fontPickerViewControllerDidCancel(_ viewController: UIFontPickerViewController)
}
```

**Самый частый и рекомендуемый паттерн 2026 года**

```swift
class FontPickerExampleViewController: UIViewController, UIFontPickerViewControllerDelegate {
    
    private let previewLabel = UILabel()
    private var selectedFont: UIFont = .preferredFont(forTextStyle: .body)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        previewLabel.text = "Пример текста в выбранном шрифте\nAaBbCc 123"
        previewLabel.numberOfLines = 0
        previewLabel.textAlignment = .center
        previewLabel.font = selectedFont.withSize(28)
        previewLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(previewLabel)
        
        NSLayoutConstraint.activate([
            previewLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            previewLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -60)
        ])
        
        let button = UIButton(type: .system)
        button.setTitle("Выбрать шрифт", for: .normal)
        button.titleLabel?.font = .preferredFont(forTextStyle: .headline)
        button.addTarget(self, action: #selector(openFontPicker), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            button.topAnchor.constraint(equalTo: previewLabel.bottomAnchor, constant: 40),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    @objc private func openFontPicker() {
        let config = UIFontPickerViewController.Configuration()
        config.includeFaces = true                  // показывать Regular, Bold, Italic и т.д.
        config.filterPredicate = UIFontPickerViewController.Configuration.defaultFilterPredicate
        // или только русскоязычные шрифты:
        // config.filterPredicate = UIFontDescriptor.filterPredicate(matchingLanguages: ["ru"])
        
        let picker = UIFontPickerViewController()
        picker.configuration = config
        picker.delegate = self
        
        present(picker, animated: true)
    }
    
    // Вызывается каждый раз при выборе шрифта (live-preview)
    func fontPickerViewControllerDidPickFont(_ viewController: UIFontPickerViewController) {
        if let descriptor = viewController.selectedFontDescriptor {
            // Самый надёжный способ получить UIFont из дескриптора
            let font = UIFont(descriptor: descriptor, size: 0) // size: 0 = сохранить размер дескриптора
            
            // Или явно задать размер
            // let font = UIFont(descriptor: descriptor, size: 28)
            
            selectedFont = font
            previewLabel.font = font.withSize(28)
            
            print("Выбран шрифт:", font.fontName)
        }
    }
    
    // Пользователь нажал Done
    func fontPickerViewControllerDidCancel(_ viewController: UIFontPickerViewController) {
        viewController.dismiss(animated: true)
        print("Выбор шрифта отменён")
    }
}
```

### Полезные фильтры (Configuration.filterPredicate)

```swift
// Только фиксированной ширины (monospaced) — для кода
UIFontPickerViewController.Configuration.filterPredicate(for: .monospaced)

// Только с засечками (serif)
UIFontPickerViewController.Configuration.filterPredicate(for: .serif)

// Только без засечек (sans-serif)
UIFontPickerViewController.Configuration.filterPredicate(for: .sansSerif)

// Только шрифты с поддержкой определённого языка
UIFontDescriptor.filterPredicate(matchingLanguages: ["ru", "en"])

// Только шрифты, поддерживающие кириллицу
UIFontDescriptor.filterPredicate(matchingLanguages: ["ru"])
```

### Лучшие практики UIFontPickerViewController в 2026

- **Всегда** используйте `configuration.includeFaces = true` — пользователи ожидают видеть Bold, Italic и т.д.  
- **Для live-preview** — обновляйте UI в `fontPickerViewControllerDidPickFont`  
- **Для сохранения** — сохраняйте `UIFontDescriptor` ([[Codable]]) вместо имени шрифта — надёжнее  
- **Для русского языка** — почти всегда добавляйте фильтр `matchingLanguages: ["ru"]`  
- **Для [[SwiftUI]]** — используйте нативный `UIFontPicker` (iOS 14+) — он проще и не требует делегата  
- **Privacy Manifest** — не требуется (это UI-компонент, не использует приватные данные)  
- **Документируйте** — пишите комментарий «UIFontPickerViewController — системный выбор шрифта с поддержкой всех начертаний и фильтром по кириллице»

**Короткий итог 2026**:
> UIFontPickerViewController — это **нативный системный пикер шрифтов** из приложения «Шрифты».  
> В 2026 году:  
> - главный метод делегата — `fontPickerViewControllerDidPickFont` (live-выбор)  
> - конфигурация через `UIFontPickerViewController.Configuration` (фильтры, faces)  
> - идеально для: текстовых редакторов, заметок, дизайнерских приложений  
> - это **единственный рекомендуемый** способ дать пользователю выбрать любой установленный шрифт  
