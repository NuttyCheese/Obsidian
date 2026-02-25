**UIMenuBuilder** — это протокол в [[UIKit]] (доступен с **iOS 13**, 2019), который позволяет **динамически строить** и **кастомизировать** меню команд ([[UIMenu]]), включая:

- главное меню приложения (Main Menu в macOS Catalyst и iPadOS)
- контекстные меню (edit menu при выделении текста)
- меню кнопок в toolbar / navigation bar
- меню в других местах, где используется `UIMenu`

С помощью `UIMenuBuilder` вы можете:

- добавлять/удалять/переупорядочивать пункты меню
- добавлять подменю
- реагировать на состояние приложения (выделен ли текст, есть ли clipboard и т.д.)
- поддерживать **стандартные системные команды** (Copy, Paste, Undo, Redo, Select All и т.д.)

Это **самый мощный и рекомендуемый** способ в 2026 году кастомизировать меню команд в UIKit-приложениях, особенно если вы поддерживаете **macOS Catalyst** или **iPadOS с клавиатурой**.

### Основные методы протокола UIMenuBuilder

| Метод делегата                                      | Что делает                                                                 | Когда вызывается                                      | Самый частый сценарий |
|-----------------------------------------------------|-----------------------------------------------------------------------------|--------------------------------------------------------|-----------------------|
| `buildMenu(with:)`                                  | Главный метод — здесь вы строите всё меню                                  | Когда система запрашивает меню (при открытии меню-бара или контекстного меню) | Добавление кастомных команд |
| `menu(for:)` (опционально)                          | Возвращает конкретное меню по идентификатору                                | Когда нужно переопределить системное меню              | Кастомизация Edit Menu |
| `canPerformAction(_:withSender:)` (опционально)     | Определяет, можно ли выполнить действие                                    | При проверке доступности команды                       | Отключение/включение Undo/Redo |
| `validate(_:)` (опционально)                        | Обновляет состояние команды (enabled/disabled, checked)                    | При каждом обновлении меню                             | Динамическое включение/выключение |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Добавление кастомных команд в главное меню + контекстное меню)

```swift
import UIKit

// 1. Делаем AppDelegate или SceneDelegate реализующим UIMenuBuilder
class AppDelegate: UIResponder, UIApplicationDelegate, UIMenuBuilder {
    
    func buildMenu(with builder: UIMenuBuilder) {
        super.buildMenu(with: builder)
        
        // Добавляем кастомное главное меню (File → New Note)
        builder.insertChild(newNoteMenu(), atStartOfMenu: .file)
        
        // Добавляем в Edit меню кастомный пункт
        builder.insertChild(customEditCommands(), atStartOfMenu: .edit)
    }
    
    private func newNoteMenu() -> UIMenu {
        let newNote = UIAction(title: "Новая заметка",
                               image: UIImage(systemName: "square.and.pencil")) { _ in
            NotificationCenter.default.post(name: .createNewNote, object: nil)
        }
        
        return UIMenu(title: "", options: .displayInline, children: [newNote])
    }
    
    private func customEditCommands() -> UIMenu {
        let copyWithEmoji = UIAction(title: "Копировать с 🔥") { _ in
            // кастомная логика копирования
        }
        
        return UIMenu(title: "", options: .displayInline, children: [copyWithEmoji])
    }
}

// 2. Регистрируем делегат в AppDelegate или SceneDelegate
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    UIMenuSystem.main.setMenuBuilder(self)
    return true
}
```

### Контекстное меню при выделении текста ([[UIEditMenuInteraction]])

```swift
class CustomTextView: UITextView {
    
    override func didMoveToWindow() {
        super.didMoveToWindow()
        
        // Добавляем современную интеракцию
        let interaction = UIEditMenuInteraction(delegate: self)
        addInteraction(interaction)
    }
}

extension CustomTextView: UIEditMenuInteractionDelegate {
    
    func editMenuInteraction(_ interaction: UIEditMenuInteraction,
                             menuFor configuration: UIEditMenuConfiguration,
                             at location: CGPoint) -> UIMenu? {
        
        let copyWithDate = UIAction(title: "Копировать с датой") { _ in
            let date = DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .short)
            UIPasteboard.general.string = "\(self.text ?? "") — \(date)"
        }
        
        return UIMenu(title: "", children: [
            copyWithDate,
            // стандартные команды остаются автоматически
        ])
    }
}
```

### Лучшие практики UIMenuBuilder в 2026 году

- **Глобально** регистрируйте `UIMenuBuilder` через `UIMenuSystem.main.setMenuBuilder(_:)` в `didFinishLaunchingWithOptions`  
- **Для контекстного меню** — используйте `UIEditMenuInteraction` + `UIMenu` (а не старый UIMenuController)  
- **Используйте** `insertChild(_:atStartOfMenu:)` / `insertSibling(_:afterMenu:)` — чтобы не перезаписывать системные меню  
- **Для динамики** — проверяйте состояние в `canPerformAction` / `validate`  
- **Для [[SwiftUI]]** — используйте `.menu` / `.contextMenu` — UIMenuBuilder нужен только в UIKit  
- **Для macOS Catalyst** — это **единственный** способ добавить пункты в главное меню (File, Edit, View и т.д.)  
- **Документируйте** — пишите комментарий:

```swift
/// Добавление кастомных команд в главное меню приложения
func buildMenu(with builder: UIMenuBuilder) {
    builder.insertChild(customFileMenu(), atStartOfMenu: .file)
}
```

**Короткий итог 2026**:
> `UIMenuBuilder` — протокол для **динамического построения** меню команд (главное меню, контекстное меню, toolbar menu).  
> В 2026 году:  
> - ключевой метод — `buildMenu(with:)`  
> - самый популярный сценарий — добавление пунктов в File / Edit меню на iPad / Mac Catalyst  
> - для контекстного меню при выделении — используйте `UIEditMenuInteraction` + `UIMenu`  
> - это **единственный современный** способ кастомизировать меню команд в UIKit  
