**UIMenuItem** — это устаревший класс в UIKit, который использовался для создания **пользовательских пунктов меню** в **UIMenuController** (контекстное меню при долгом нажатии на текст, изображения и т.д.).

### Важный статус на 2026 год

С iOS 13 (2019) и особенно с iOS 14+ класс **UIMenuItem** официально **устарел** (deprecated) и практически **не используется** в новых проектах.

Apple полностью заменила его на современный и гораздо более мощный механизм — **UIMenu** + **UIMenuBuilder** + **UIEditMenuInteraction** (iOS 13+ / iOS 16+).

### Краткая история и актуальность

| Версия iOS     | Статус UIMenuItem                          | Рекомендуемый способ в 2026                          | Почему перешли |
|----------------|--------------------------------------------|-------------------------------------------------------|----------------|
| iOS 3 – iOS 12 | Основной способ кастомных пунктов меню     | UIMenuItem                                            | — |
| iOS 13 – iOS 15| Deprecated, но ещё работает                | UIMenu + UIEditMenuInteraction                        | Более гибкий API |
| iOS 16+        | Deprecated, не рекомендуется               | UIEditMenuInteraction + UIMenu                        | Полная поддержка жестов, предпросмотра, динамики |
| 2026 (текущий) | Почти мёртв, использовать только в legacy  | UIEditMenuInteraction                                 | Современный стандарт |

### Как выглядел UIMenuItem (для понимания legacy-кода)

```swift
// Старый способ (до iOS 13 — не используйте в 2026!)
let copyItem = UIMenuItem(title: "Копировать с эмодзи", action: #selector(copyWithEmoji))
UIMenuController.shared.menuItems = [copyItem]
UIMenuController.shared.showMenu(from: view, rect: someRect)
```

### Современный эквивалент в 2026 году  
(UIEditMenuInteraction + UIMenu — рекомендуемый способ)

```swift
import UIKit

class CustomTextViewController: UIViewController {
    
    private let textView = UITextView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        textView.text = "Долгое нажатие → кастомное меню"
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
extension CustomTextViewController: UIEditMenuInteractionDelegate {
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        
        // Создаём кастомное меню
        let copyWithEmoji = UIAction(title: "Копировать с эмодзи 🔥") { _ in
            let pasteboard = UIPasteboard.general
            pasteboard.string = (self.textView.text ?? "") + " 🔥"
        }
        
        let share = UIAction(title: "Поделиться") { _ in
            let activityVC = UIActivityViewController(activityItems: [self.textView.text ?? ""], applicationActivities: nil)
            self.present(activityVC, animated: true)
        }
        
        return UIMenu(title: "", children: [copyWithEmoji, share])
    }
    
    // Опционально: позиция меню
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             previewForMenuAt location: CGPoint) -> UITargetedPreview? {
        // Можно вернуть кастомный предпросмотр
        return nil
    }
}
```

### Когда в 2026 году ещё можно встретить UIMenuItem

- Очень старые проекты (поддержка iOS 12 и ниже)  
- Legacy-код, который ещё не мигрировали  
- Редкие случаи, когда нужен именно старый UIMenuController (например, в очень специфичных кастомных текстовых views)

### Рекомендации по миграции в 2026 году

| Старая конструкция                        | Новый способ (iOS 13+)                                  | Преимущества |
|-------------------------------------------|----------------------------------------------------------|--------------|
| `UIMenuController.shared.menuItems`       | `UIEditMenuInteraction` + `UIMenu`                       | Гибкость, предпросмотр, жесты, динамическое меню |
| `UIMenuItem` с кастомным action           | `UIAction` внутри `UIMenu`                               | Легко добавлять подменю, атрибуты, состояния |
| Долгое нажатие → меню                     | Добавление `UIEditMenuInteraction` к UITextView / UIView | Полный контроль, поддержка новых жестов |

### Короткий итог 2026

> `UIMenuItem` — **устаревший** (deprecated с iOS 13) способ добавления пунктов в контекстное меню `UIMenuController`.  
> В 2026 году:  
> - **не используйте** в новых проектах  
> - заменяется на **UIEditMenuInteraction** + **UIMenu** + **UIAction**  
> - новый API мощнее, поддерживает подменю, предпросмотр, динамику и жесты  
> - это **единственный рекомендуемый** способ кастомизировать контекстное меню в современном UIKit  

Удачи с современными и удобными контекстными меню в твоём приложении! 📋