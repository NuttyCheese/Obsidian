**UICommand** — это класс в [[UIKit]] ([[iOS]] 13+), который представляет **команду** (действие) в системных меню, таких как:

- **Edit Menu** (контекстное меню при долгом нажатии на текст/изображение)
- **[[UIMenu]]** в UIBarButtonItem (меню в навигационной панели)
- **Shortcut Menu** (меню быстрых действий на иконке приложения)
- **Command Palette** (в macOS Catalyst и iPadOS с клавиатурой)
- **Share Sheet** (расширенные действия в меню "Поделиться")

UICommand — это **современная замена** старому [[UIMenuItem]] и частично [[UIAction]] (но они всё ещё используются вместе).

### Ключевые особенности UICommand (2025–2026)

| Свойство / Возможность          | Описание                                                                 | Когда использовать |
|---------------------------------|--------------------------------------------------------------------------|---------------------|
| title                           | Текст команды ("Copy", "Paste", "Share…")                                | Обязательно |
| image / selectedImage           | Иконка (SF Symbols или кастомная)                                        | Для визуальной привлекательности |
| action                          | Селектор метода, который будет вызван при выборе                        | Основное действие |
| propertyList                    | Любые данные (Any?), передаются в target-action                         | Для передачи контекста |
| attributes                      | .disabled, .destructive, .hidden, .keepsMenuPresented                   | Управление состоянием |
| state                           | .on / .off / .mixed (чекбокс/радио-кнопка)                              | Для toggle-команд |
| discoverabilityTitle            | Подсказка в Command Palette (macOS/iPad с клавиатурой)                  | Улучшает доступность |
| input                           | Клавиатурный шорткат (например, "c" для Copy)                           | Для power users |
| modifierFlags                   | .command, .shift, .option, .control                                     | Комбинации клавиш |

### Самые популярные паттерны использования в 2026 году

#### 1. Добавление кастомной команды в Edit Menu (текст)

```swift
class TextViewController: UIViewController, UITextViewDelegate {
    
    @IBOutlet private weak var textView: UITextView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        textView.delegate = self
    }
    
    // Перехватываем системное меню редактирования
    override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
        if action == #selector(customTranslate) {
            return textView.selectedTextRange != nil
        }
        return super.canPerformAction(action, withSender: sender)
    }
    
    @objc private func customTranslate() {
        guard let selected = textView.selectedText else { return }
        print("Переводим: \(selected)")
        // Здесь вызов API перевода
    }
    
    // Более современный способ — через UIMenu (iOS 13+)
    override func buildMenu(with builder: UIMenuBuilder) {
        super.buildMenu(with: builder)
        
        let translateCommand = UICommand(
            title: "Перевести",
            image: UIImage(systemName: "globe"),
            action: #selector(customTranslate),
            propertyList: nil
        )
        
        let translateMenu = UIMenu(
            title: "",
            options: .displayInline,
            children: [translateCommand]
        )
        
        builder.insertChild(translateMenu, atStartOfMenu: .edit)
    }
}
```

#### 2. Создание кастомного меню в [[UINavigationBar]] / [[UIToolbar]]

```swift
let shareCommand = UICommand(
    title: "Поделиться",
    image: UIImage(systemName: "square.and.arrow.up"),
    action: #selector(shareTapped),
    propertyList: ["type": "link"]
)

let deleteCommand = UICommand(
    title: "Удалить",
    image: UIImage(systemName: "trash"),
    action: #selector(deleteTapped),
    attributes: .destructive
)

let menu = UIMenu(children: [shareCommand, deleteCommand])

let barButton = UIBarButtonItem(title: "Действия", menu: menu)
navigationItem.rightBarButtonItem = barButton
```

#### 3. Toggle-команда (чекбокс в меню)

```swift
let favoriteCommand = UICommand(
    title: "В избранное",
    image: UIImage(systemName: "star"),
    action: #selector(toggleFavorite),
    state: isFavorite ? .on : .off
)
```

### Лучшие практики UICommand в Swift 2026

- **Предпочитайте** UICommand + UIMenu вместо старых UIMenuItem (они deprecated)  
- **Используйте** `buildMenu(with:)` для добавления команд в системные меню (Edit Menu, Share Sheet)  
- **Для кнопок в навигации** — используйте `UIBarButtonItem(menu:)`  
- **Для SF Symbols** — всегда используйте `UIImage(systemName:)` с правильным weight (.medium / .semibold)  
- **Для деструктивных действий** — добавляйте атрибут `.destructive`  
- **Для шорткатов** — задавайте `input` и `modifierFlags` для пользователей с клавиатурой  
- **В SwiftUI** — используйте `Menu` / `ContextMenu` — они построены поверх UICommand  
- **Документируйте** — пишите комментарий «UICommand — кастомная команда "Перевести" в Edit Menu с иконкой и селектором»

**Короткий итог 2026**:
> UICommand — это **современный объект команды** для меню в UIKit (Edit Menu, UIBarButtonItem menu, Command Palette).  
> В 2026 году:  
> - создаётся через `UICommand(title:action:)`  
> - добавляется в UIMenu и привязывается к UIBarButtonItem / buildMenu(with:)  
> - поддерживает иконки, состояния (.on/.off), атрибуты (.destructive), шорткаты  
> - это **единственный рекомендуемый** способ создавать кастомные действия в системных меню UIKit  
