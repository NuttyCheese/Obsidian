**UIAlertController** — это системный контроллер в [[UIKit]], который используется для отображения **всплывающих окон** (alerts, action sheets, confirmation dialogs, login prompts и т.д.) в [[iOS]]-приложениях.

Это **единственный рекомендуемый** способ показывать модальные диалоги в UIKit-приложениях в 2026 году.

### Основные типы UIAlertController

| Тип (style)                | Внешний вид                                      | Когда использовать в 2026 году                          | Максимум кнопок |
|----------------------------|--------------------------------------------------|----------------------------------------------------------|-----------------|
| `.alert`                   | Классическое окно с заголовком, сообщением и кнопками по центру | Ошибки, предупреждения, подтверждения действия, ввод текста | 2 (по Human Interface Guidelines) |
| `.actionSheet`             | Лист действий, выезжающий снизу (или popover на iPad) | Выбор действия из нескольких вариантов, меню «Ещё…»     | До 10–12 (но Apple рекомендует ≤6) |

### Самый важный принцип 2026 года

> Apple Human Interface Guidelines:  
> - Используй **alert** только для **важных, критических** сообщений, которые требуют немедленного внимания пользователя.  
> - Всё остальное — **action sheet**, **[[UIMenu]]**, **context menu** или **[[SwiftUI]] sheet/popover**.

### Самый современный и рекомендуемый паттерн 2026 года

#### 1. Простое предупреждение с одной кнопкой (OK)

```swift
func showSimpleAlert() {
    let alert = UIAlertController(
        title: "Добро пожаловать!",
        message: "Это ваше первое использование приложения.",
        preferredStyle: .alert
    )
    
    alert.addAction(UIAlertAction(title: "OK", style: .default) { _ in
        print("Пользователь нажал OK")
    })
    
    present(alert, animated: true)
}
```

#### 2. Полноценный alert с двумя кнопками (Подтвердить / Отмена)

```swift
func showDeleteConfirmation() {
    let alert = UIAlertController(
        title: "Удалить задачу?",
        message: "Это действие нельзя отменить.",
        preferredStyle: .alert
    )
    
    alert.addAction(UIAlertAction(title: "Отмена", style: .cancel) { _ in
        print("Удаление отменено")
    })
    
    alert.addAction(UIAlertAction(title: "Удалить", style: .destructive) { _ in
        // Удаление задачи
        deleteTask()
    })
    
    present(alert, animated: true)
}
```

#### 3. Action Sheet (лист действий) — самый частый в 2026

```swift
func showShareOptions() {
    let actionSheet = UIAlertController(
        title: "Поделиться фото",
        message: nil,
        preferredStyle: .actionSheet
    )
    
    actionSheet.addAction(UIAlertAction(title: "Сохранить в Фото", style: .default) { _ in
        saveToPhotos()
    })
    
    actionSheet.addAction(UIAlertAction(title: "Отправить в Telegram", style: .default) { _ in
        shareToTelegram()
    })
    
    actionSheet.addAction(UIAlertAction(title: "Отмена", style: .cancel))
    
    // Важно для iPad: указать sourceView / sourceRect
    if let popover = actionSheet.popoverPresentationController {
        popover.sourceView = shareButton
        popover.sourceRect = shareButton.bounds
        popover.permittedArrowDirections = .any
    }
    
    present(actionSheet, animated: true)
}
```

#### 4. Alert с текстовым полем (ввод имени, пароля и т.д.)

```swift
func showLoginPrompt() {
    let alert = UIAlertController(
        title: "Введите имя",
        message: "Как вас зовут?",
        preferredStyle: .alert
    )
    
    alert.addTextField { textField in
        textField.placeholder = "Ваше имя"
        textField.autocapitalizationType = .words
    }
    
    alert.addAction(UIAlertAction(title: "Отмена", style: .cancel))
    
    alert.addAction(UIAlertAction(title: "Сохранить", style: .default) { _ in
        guard let name = alert.textFields?.first?.text, !name.isEmpty else { return }
        saveUserName(name)
    })
    
    present(alert, animated: true)
}
```

### Лучшие практики UIAlertController в Swift 2026

- **title** — короткий и понятный (1–2 слова)  
- **message** — подробное объяснение, но не длиннее 3–4 строк  
- **Стили кнопок**:
  - `.default` — обычное действие  
  - `.cancel` — отмена (всегда последняя)  
  - `.destructive` — опасное действие (красный цвет)  
- **iPad / Mac Catalyst** — **обязательно** указывай `popoverPresentationController.sourceView` и `sourceRect`, иначе краш  
- **Не используй alert для простых уведомлений** — используй `UIMenu`, `UIAlertController.alert` только для важных решений  
- **Async/await** — презентуй на `@MainActor`  
- **Swift 6 strict concurrency** — UIAlertController полностью безопасен  
- **Документируйте** — пиши комментарий «UIAlertController — подтверждение удаления с двумя кнопками»

**Короткий девиз 2026**:
> UIAlertController — это **системный диалог** Apple: выглядит как родной, обновляется с каждой iOS, поддерживает все возможности (текст, кнопки, ввод).  
> В 2026 году используй **.alert** только для важных решений, **.actionSheet** — для выбора действия.  
> Всегда указывай sourceView/sourceRect на iPad и не злоупотребляй — пользователь устал от лишних алертов.
