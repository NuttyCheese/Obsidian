**UIBarButtonItem** — это специальный элемент управления в UIKit, предназначенный исключительно для размещения в двух местах:

- **UINavigationBar** (навигационная панель сверху)
- **UIToolbar** (панель инструментов снизу или кастомная)

Это **не** обычная `UIButton` и **не** наследуется от `UIView`.  
Это подкласс `UIBarItem`, который выглядит и ведёт себя как кнопка, но управляется через специальный API навигации и тулбара.

В 2026 году это **основной** способ добавить кнопки в навигацию и нижнюю панель инструментов в UIKit-приложениях.

### Основные типы UIBarButtonItem

| Тип создания                                 | Внешний вид / поведение                              | Когда использовать в 2026 | Пример |
|----------------------------------------------|-------------------------------------------------------|----------------------------|--------|
| **Текстовый** (title)                        | Просто текст («Сохранить», «Готово»)                  | Простые действия, где иконка не нужна | `UIBarButtonItem(title: "Сохранить", style: .plain, ...)` |
| **Иконка** (image)                           | SF Symbol или кастомная картинка                      | Современный минималистичный дизайн | `UIBarButtonItem(image: UIImage(systemName: "plus"), ...)` |
| **Системная кнопка** (barButtonSystemItem)   | Готовые иконки Apple (edit, add, done, trash и т.д.)  | Стандартные действия (редактировать, добавить, отменить) | `UIBarButtonItem(barButtonSystemItem: .edit, ...)` |
| **Кастомная вью** (customView)               | Любая UIView (UILabel, UIImageView, UIStackView и т.д.) | Сложный UI: аватар, счётчик, прогресс | `UIBarButtonItem(customView: customAvatarView)` |

### Стили (style) — влияют на внешний вид текста

| Стиль               | Внешний вид текста                          | Когда использовать в 2026 |
|---------------------|---------------------------------------------|----------------------------|
| `.plain`            | Обычный текст (синий на светлой теме)       | Большинство кнопок         |
| `.done`             | Жирный, синий (или акцентный) текст         | Кнопка «Готово», «Сохранить» в формах |
| `.bordered` (устарел) | Рамка вокруг текста (iOS < 13)              | Не используйте — устарел   |

### Самый популярный и современный паттерн 2026 года

#### 1. Кнопка в Navigation Bar (самый частый случай)

```swift
class DetailViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Кнопка "Сохранить" справа
        let saveButton = UIBarButtonItem(
            title: "Сохранить",
            style: .done,
            target: self,
            action: #selector(saveTapped)
        )
        navigationItem.rightBarButtonItem = saveButton
        
        // Кнопка "Добавить" слева с иконкой
        let addButton = UIBarButtonItem(
            image: UIImage(systemName: "plus.circle.fill"),
            style: .plain,
            target: self,
            action: #selector(addTapped)
        )
        navigationItem.leftBarButtonItem = addButton
    }
    
    @objc private func saveTapped() {
        print("Сохранение...")
    }
    
    @objc private func addTapped() {
        print("Добавление...")
    }
}
```

#### 2. Несколько кнопок справа (группа действий)

```swift
let editButton = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))
let shareButton = UIBarButtonItem(barButtonSystemItem: .action, target: self, action: #selector(shareTapped))

navigationItem.rightBarButtonItems = [shareButton, editButton]
```

#### 3. Кнопка с кастомной вью (аватар, счётчик, прогресс)

```swift
let avatarView = UIImageView(image: UIImage(systemName: "person.circle.fill"))
avatarView.contentMode = .scaleAspectFill
avatarView.frame = CGRect(x: 0, y: 0, width: 32, height: 32)
avatarView.layer.cornerRadius = 16
avatarView.clipsToBounds = true

let avatarButton = UIBarButtonItem(customView: avatarView)

// Добавляем tap gesture вручную
let tap = UITapGestureRecognizer(target: self, action: #selector(profileTapped))
avatarView.isUserInteractionEnabled = true
avatarView.addGestureRecognizer(tap)

navigationItem.rightBarButtonItem = avatarButton
```

#### 4. Кнопки в UIToolbar (нижняя панель)

```swift
let toolbar = UIToolbar(frame: CGRect(x: 0, y: view.bounds.height - 50, width: view.bounds.width, height: 50))

let flexibleSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
let doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneTapped))
let cancelButton = UIBarButtonItem(barButtonSystemItem: .cancel, target: self, action: #selector(cancelTapped))

toolbar.items = [cancelButton, flexibleSpace, doneButton]
view.addSubview(toolbar)
```

### Лучшие практики UIBarButtonItem в Swift 2026

- **Используй UIAction вместо target-action** (iOS 14+)

```swift
let saveAction = UIAction { _ in
    print("Сохранено")
}

let saveButton = UIBarButtonItem(title: "Сохранить", primaryAction: saveAction)
navigationItem.rightBarButtonItem = saveButton
```

- **Для системных иконок** — всегда используй `barButtonSystemItem` (`.edit`, `.add`, `.done`, `.trash`, `.action` и т.д.) — они автоматически адаптируются под тему и доступность  
- **iPad / Mac Catalyst** — обязательно указывай `popoverPresentationController` при показе action sheet из кнопки  
- **SF Symbols** — используй `UIImage(systemName:weight:scale:)` для иконок  
- **Не делай слишком много кнопок справа** — максимум 2–3, иначе интерфейс перегружен  
- **@MainActor** — все действия кнопок — на главном акторе  
- **Swift 6 strict concurrency** — UIBarButtonItem полностью безопасен  
- **Документируйте** — пиши комментарий «UIBarButtonItem — кнопка "Сохранить" в навигационной панели»

**Короткий девиз 2026**:
> UIBarButtonItem — это **кнопка специально для навигационной панели и тулбара**.  
> В 2026 году используй **UIAction** вместо target-action, системные иконки и максимум 2–3 кнопки справа.  
> Это **единственный правильный** способ добавить действия в верхнюю/нижнюю панель в UIKit.

Удачи с чистыми и нативными навигационными панелями в твоём приложении! 🔝