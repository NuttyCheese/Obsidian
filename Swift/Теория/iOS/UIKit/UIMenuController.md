**UIMenuController** — это устаревший (==deprecated==) системный контроллер в [[UIKit]], который отвечал за отображение **контекстного меню** при долгом нажатии на текст, изображения или другие элементы (копировать, вырезать, вставить, выделить всё и т.д.).

### Текущий статус на 2026 год

| Версия iOS      | Статус UIMenuController                       | Рекомендуемый современный способ                | Почему перешли                                            |
| --------------- | --------------------------------------------- | ----------------------------------------------- | --------------------------------------------------------- |
| iOS 3 – iOS 12  | Основной и единственный способ                | UIMenuController                                | —                                                         |
| iOS 13 – iOS 15 | ==Deprecated==, но ещё работает               | [[UIEditMenuInteraction]] + [[UIMenu]]          | Более мощный и гибкий [[API]]                             |
| iOS 16 – 2026   | ==Deprecated==, не рекомендуется, почти мёртв | UIEditMenuInteraction + UIEditMenuConfiguration | Полная поддержка жестов, предпросмотра, динамики, подменю |

**Коротко**:  
в 2026 году **UIMenuController почти не используется** в новых проектах.  
Apple полностью заменила его на **UIEditMenuInteraction** (iOS 13+) + **[[UIMenu]]** + **[[UIAction]]**.

### Как выглядел UIMenuController (только для понимания старого кода)

```swift
// Старый код (до iOS 13 — НЕ ИСПОЛЬЗУЙТЕ в 2026!)
class OldTextViewController: UIViewController, UITextViewDelegate {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let textView = UITextView(frame: view.bounds)
        textView.text = "Долгое нажатие → старое меню"
        view.addSubview(textView)
    }
    
    // Добавляем кастомный пункт меню
    override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
        if action == #selector(customCopy(_:)) {
            return true
        }
        return super.canPerformAction(action, withSender: sender)
    }
    
    @objc func customCopy(_ sender: Any?) {
        print("Копировать с эмодзи!")
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        // Добавляем кастомный пункт
        let customItem = UIMenuItem(title: "Копировать с 🔥", action: #selector(customCopy(_:)))
        UIMenuController.shared.menuItems = [customItem]
    }
}
```

### Современный эквивалент в 2026 году  
(UIEditMenuInteraction + UIMenu — единственный рекомендуемый способ)

```swift
import UIKit

class ModernTextViewController: UIViewController {
    
    private let textView = UITextView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        textView.text = "Долгое нажатие → современное меню"
        textView.isEditable = true
        textView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(textView)
        
        NSLayoutConstraint.activate([
            textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            textView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            textView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            textView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
        
        // 1. Создаём интеракцию для кастомного меню
        let interaction = UIEditMenuInteraction(delegate: self)
        textView.addInteraction(interaction)
    }
}

// MARK: - UIEditMenuInteractionDelegate
extension ModernTextViewController: UIEditMenuInteractionDelegate {
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        
        // Кастомные действия
        let copyWithEmoji = UIAction(title: "Копировать с 🔥") { _ in
            let pasteboard = UIPasteboard.general
            pasteboard.string = (self.textView.text ?? "") + " 🔥"
        }
        
        let share = UIAction(title: "Поделиться", image: UIImage(systemName: "square.and.arrow.up")) { _ in
            let activityVC = UIActivityViewController(activityItems: [self.textView.text ?? ""], applicationActivities: nil)
            self.present(activityVC, animated: true)
        }
        
        let delete = UIAction(title: "Удалить", image: UIImage(systemName: "trash"), attributes: .destructive) { _ in
            self.textView.text = ""
        }
        
        return UIMenu(title: "", options: .displayInline, children: [
            copyWithEmoji,
            share,
            delete
        ])
    }
    
    // Опционально: предпросмотр при долгом нажатии
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        // Можно вернуть кастомный предпросмотр текста
        return nil
    }
}
```

### Когда в 2026 году ещё можно встретить UIMenuController

- Очень старые проекты (поддержка iOS 12 и ниже)  
- Legacy-код, который ещё не мигрировали  
- Редкие случаи, когда нужен именно старый UIMenuController (например, в очень специфичных текстовых views)

### Рекомендации по миграции в 2026 году

| Старая конструкция                        | Новый способ (iOS 13+)                                  | Преимущества |
|-------------------------------------------|----------------------------------------------------------|--------------|
| `UIMenuController.shared.menuItems`       | `UIEditMenuInteraction` + `UIMenu`                       | Поддержка подменю, предпросмотра, динамики |
| `UIMenuItem` с кастомным action           | `UIAction` внутри `UIMenu`                               | Легко добавлять состояния, атрибуты, изображения |
| Долгое нажатие → меню                     | Добавление `UIEditMenuInteraction` к UITextView / UIView | Современный UX, поддержка новых жестов |

**Короткий итог 2026**:
> `UIMenuController` — **устаревший** (deprecated с iOS 13) контроллер для контекстного меню при долгом нажатии.  
> В 2026 году:  
> - **не используйте** в новых проектах  
> - заменяется на **UIEditMenuInteraction** + **UIMenu** + **UIAction**  
> - новый API мощнее, красивее и поддерживает предпросмотр, подменю, динамику  
> - это **единственный рекомендуемый** способ кастомизировать контекстное меню в современном UIKit  
