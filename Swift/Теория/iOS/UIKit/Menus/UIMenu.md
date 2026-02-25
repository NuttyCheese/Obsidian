**UIMenu** — это класс в [[UIKit]] (появился в iOS 13, 2019), который описывает **иерархическое меню** — современную замену старым [[UIMenuItem]] и [[UIAlertController]]-style action sheets.

UIMenu используется везде, где Apple показывает меню с действиями:

- контекстное меню при долгом нажатии (Edit Menu)
- меню кнопки в навигационной панели ([[UIBarButtonItem]].menu)
- меню быстрых действий на иконке приложения ([[UIApplicationShortcutItem]])
- меню в Command Palette (iPadOS + клавиатура)
- меню «Поделиться» (расширенные действия)

Это **самый рекомендуемый** способ в 2025–2026 годах создавать любые меню в UIKit-приложениях.

### Ключевые особенности UIMenu (актуально 2026)

| Свойство / Возможность               | Тип / Значение     | Что делает / зачем нужен                                 | Самый частый сценарий               |
| ------------------------------------ | ------------------ | -------------------------------------------------------- | ----------------------------------- |
| `title`                              | [[String]]?        | Заголовок меню (может быть пустым)                       | Подсказка или категория             |
| `subtitle`                           | String? (iOS 15+)  | Подзаголовок (серый текст под title)                     | Дополнительная информация           |
| `image`                              | [[UIImage]]?       | Иконка слева от заголовка                                | SF Symbols, кастомные               |
| `options`                            | UIMenu.Options     | `.displayInline`, `.destructive`, `.restrictive` и др.   | `.displayInline` — самый популярный |
| `children`                           | [UIMenuElement]    | Массив действий ([[UIAction]] или вложенные UIMenu)      | Основное содержимое                 |
| `identifier`                         | String?            | Уникальный идентификатор для обновления меню динамически | Динамическое обновление             |
| `preferredElementSize` ([[iOS]] 16+) | UIMenu.ElementSize | `.medium`, `.large`, `.automatic`                        | Адаптация под iPad/macOS            |

### Основные типы UIMenuElement

| Класс                                   | Когда использовать                                  | Пример использования               |
| --------------------------------------- | --------------------------------------------------- | ---------------------------------- |
| **UIAction**                            | Одиночное действие (нажатие → селектор или closure) | «Копировать», «Удалить»            |
| **UIMenu**                              | Вложенное подменю                                   | «Поделиться → Telegram / WhatsApp» |
| **[[UIDeferredMenuElement]]** (iOS 14+) | Лениво загружаемое меню (асинхронно)                | Меню с данными из сети             |

### Самые популярные паттерны UIMenu в 2026 году

#### 1. Простое меню для UIBarButtonItem (самый частый)

```swift
let copyAction = UIAction(title: "Копировать", image: UIImage(systemName: "doc.on.doc")) { _ in
    UIPasteboard.general.string = "Текст для копирования"
}

let shareAction = UIAction(title: "Поделиться", image: UIImage(systemName: "square.and.arrow.up")) { _ in
    // share sheet
}

let deleteAction = UIAction(title: "Удалить", image: UIImage(systemName: "trash"), attributes: .destructive) { _ in
    // удаление
}

let menu = UIMenu(title: "Действия", children: [copyAction, shareAction, deleteAction])

let button = UIBarButtonItem(title: "Меню", image: UIImage(systemName: "ellipsis.circle"), menu: menu)
navigationItem.rightBarButtonItem = button
```

#### 2. Вложенное меню (подменю)

```swift
let telegram = UIAction(title: "Telegram") { _ in /* ... */ }
let whatsapp = UIAction(title: "WhatsApp") { _ in /* ... */ }

let shareMenu = UIMenu(title: "Поделиться в", children: [telegram, whatsapp])

let mainMenu = UIMenu(children: [
    UIAction(title: "Сохранить", image: UIImage(systemName: "arrow.down.circle")),
    shareMenu,
    UIAction(title: "Удалить", attributes: .destructive) { _ in /* ... */ }
])

button.menu = mainMenu
```

#### 3. Inline-меню (без заголовка, просто список действий)

```swift
let menu = UIMenu(options: .displayInline, children: [
    UIAction(title: "Редактировать", image: UIImage(systemName: "pencil")) { _ in },
    UIAction(title: "Удалить", image: UIImage(systemName: "trash"), attributes: .destructive) { _ in }
])
```

#### 4. Динамическое обновление меню (очень популярный паттерн 2026)

```swift
class ViewController: UIViewController {
    
    private var isFavorite = false
    
    private lazy var button: UIBarButtonItem = {
        let btn = UIBarButtonItem(title: "Действия", menu: makeMenu())
        return btn
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        navigationItem.rightBarButtonItem = button
    }
    
    private func makeMenu() -> UIMenu {
        let favoriteTitle = isFavorite ? "Убрать из избранного" : "В избранное"
        let favoriteImage = isFavorite ? "star.fill" : "star"
        
        let favoriteAction = UIAction(title: favoriteTitle, image: UIImage(systemName: favoriteImage)) { [weak self] _ in
            self?.isFavorite.toggle()
            // Обновляем меню динамически
            self?.navigationItem.rightBarButtonItem?.menu = self?.makeMenu()
        }
        
        return UIMenu(children: [favoriteAction])
    }
}
```

### Лучшие практики UIMenu в Swift 2026

- **Предпочитайте** `UIMenu` + `UIAction` вместо старых `UIMenuItem` и `UIAlertAction`  
- **Используйте** `.displayInline` для плоских списков без заголовка (выглядит чище)  
- **Для деструктивных действий** — всегда добавляйте `.destructive` (красный цвет)  
- **Для иконок** — используйте SF Symbols с правильным weight (.medium / .semibold)  
- **Для динамического меню** — пересоздавайте UIMenu и присваивайте заново `menu = newMenu`  
- **Для SwiftUI** — используйте `Menu { ... }` — он построен поверх UIMenu под капотом  
- **Для клавиатурных шорткатов** — добавляйте `input` и `modifierFlags` в UIAction  
- **Документируйте** — пишите комментарий «UIMenu — контекстное меню для кнопки с действиями "Копировать", "Поделиться", "Удалить"»

**Короткий итог 2026**:
> UIMenu — это **современный объект меню** в UIKit (iOS 13+).  
> В 2026 году:  
> - создаётся через `UIMenu(children: [UIAction])`  
> - привязывается к `UIBarButtonItem.menu`, `buildMenu(with:)`, контекстному меню  
> - поддерживает вложенность, иконки, состояния, деструктивные действия, шорткаты  
> - это **единственный рекомендуемый** способ делать меню в UIKit-приложениях  
