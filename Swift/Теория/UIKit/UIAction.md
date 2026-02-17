**UIAction** — это современный объект в UIKit (с iOS 13, 2019), который заменяет старый target-action механизм (`addTarget:action:for:`) и делает работу с событиями кнопок, меню, жестов и других элементов управления **гораздо чище, типобезопаснее и удобнее**.

Это **замыкание + метаданные** в одном объекте, а не разрозненные `#selector` + `@objc func`.

### Когда и зачем использовать UIAction (2026 реальность)

| Сценарий                                      | Старый способ (до UIAction)                     | Новый способ с UIAction (рекомендуемый 2026) | Почему UIAction лучше |
|-----------------------------------------------|--------------------------------------------------|-----------------------------------------------|-----------------------|
| Нажатие кнопки (основное действие)            | `addTarget(self, action: #selector(tapped), for: .touchUpInside)` | `button.addAction(UIAction { _ in ... }, for: .touchUpInside)` | Нет `#selector`, нет `@objc`, нет retain cycle |
| Контекстное меню (UIMenu)                     | `UIMenu(title: "", children: [UICommand(...)])` с `#selector` | `UIAction(title: "Copy", handler: { _ in ... })` | Полностью замыкание, проще, типобезопасно |
| Кастомные жесты (UIGestureRecognizer)         | `addTarget(self, action: #selector(handlePan))` | `UIGestureRecognizer.addAction(UIAction { ... })` | То же самое — чистый код |
| Кнопка в navigation bar / toolbar             | `UIBarButtonItem(target: self, action: #selector(...))` | `UIBarButtonItem(title: "Save", primaryAction: UIAction { ... })` | Нет слабых ссылок, нет утечек |
| Динамическое меню (contextMenuConfiguration)  | `#selector` + `canPerformAction`                 | `UIAction` напрямую в `UIContextMenuConfiguration` | Всё в одном месте |

**Коротко**:  
UIAction — это **замыкание**, которое можно привязать к любому событию `UIControl` или использовать в меню/контексте.  
В 2026 году это **основной** способ работы с действиями в UIKit.

### Полный синтаксис и варианты (2026 стандарт)

#### 1. Базовый UIAction (самый частый)

```swift
button.addAction(UIAction { action in
    print("Кнопка нажата!")
}, for: .touchUpInside)
```

- `action` — это сам объект UIAction (можно получить title, sender и т.д.)
- Замыкание захватывает `[weak self]` автоматически, если нужно

#### 2. С передачей sender (кнопки)

```swift
button.addAction(UIAction { [weak self] action in
    guard let self else { return }
    guard let sender = action.sender as? UIButton else { return }
    
    sender.setTitle("Нажато!", for: .normal)
}, for: .touchUpInside)
```

#### 3. С title и image (для UIMenu / context menu)

```swift
let copyAction = UIAction(title: "Копировать", image: UIImage(systemName: "doc.on.doc")) { _ in
    UIPasteboard.general.string = "Скопированный текст"
}

let menu = UIMenu(title: "", children: [copyAction])
button.showsMenuAsPrimaryAction = true
button.menu = menu
```

#### 4. С атрибутами (disabled, destructive и т.д.)

```swift
let deleteAction = UIAction(title: "Удалить",
                            image: UIImage(systemName: "trash"),
                            attributes: .destructive) { _ in
    // удаление
}

let editAction = UIAction(title: "Редактировать",
                          attributes: [.hidden]) { _ in
    // скрыто, но может быть показано позже
}
```

#### 5. В UIBarButtonItem (самый чистый способ 2026)

```swift
let saveButton = UIBarButtonItem(title: "Сохранить",
                                 primaryAction: UIAction { [weak self] _ in
    self?.saveDocument()
})
navigationItem.rightBarButtonItem = saveButton
```

### Преимущества UIAction над старым target-action

| Старый способ (`addTarget:action:`)          | UIAction (новый)                                     | Почему лучше в 2026 |
|----------------------------------------------|------------------------------------------------------|---------------------|
| Требует `@objc` и `#selector`                | Замыкание — обычная функция без `@objc`              | Меньше boilerplate  |
| Сильный захват target → retain cycle         | `[weak self]` в замыкании — безопасно                | Нет утечек памяти   |
| Метод должен быть в классе                   | Замыкание может быть где угодно                      | Гибкость            |
| Сложно передать параметры                    | Легко: `UIAction { [weak self] in self?.do(param) }` | Читаемо             |
| Трудно тестировать                           | Замыкание — легко мокать и тестировать              | Лучше для unit-тестов |

### Лучшие практики UIAction в Swift 2026

- **Всегда используй `[weak self]`** в замыканиях — это предотвращает retain cycle  
- **Используй `primaryAction`** для UIBarButtonItem и UIButton вместо старого target-action  
- **Не смешивай старый и новый стиль** в одном проекте — выбирай UIAction везде, где возможно  
- **Для меню** — всегда создавай `UIMenu(children: [UIAction(...)])`  
- **Документируйте** — пиши комментарий «UIAction — основное действие кнопки "Сохранить"»

**Короткий девиз 2026**:
> UIAction — это **замыкание + метаданные** для любого действия в UIKit.  
> В 2026 году это **единственный рекомендуемый** способ привязывать действия к кнопкам, меню, жестам и UIBarButtonItem.  
> Забудь `#selector` и `@objc func` для новых кнопок — пиши `UIAction { ... }`.

Удачи с чистым и современным кодом в Swift! 🎛️