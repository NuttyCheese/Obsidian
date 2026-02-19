**UILongPressGestureRecognizer** — это жест из [[UIKit]], который распознаёт **долгое нажатие** (long press) на любом [[UIView]] (в том числе на ячейках таблиц/коллекций, иконках, изображениях и т.д.).

Он активируется, когда пользователь удерживает палец на экране **дольше** заданного времени (по умолчанию 0.5 секунды).

### Для чего используют UILongPressGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                            | Как именно используется                                         | Пример в приложениях 2026                              |
| --------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| Контекстное меню при долгом нажатии                 | Показать [[UIMenu]] / [[UIAlertController]] / кастомный поповер | Фото в галерее, сообщение в чате, иконка в Home Screen |
| Drag & Drop (перетаскивание)                        | Начать drag после долгого нажатия                               | Перемещение задач в Todo-листе, перестановка виджетов  |
| Редактирование / удаление                           | Показать кнопки «Удалить», «Редактировать», «Переместить»       | Долгое нажатие на ячейке таблицы → swipe actions       |
| Выделение нескольких элементов                      | Включить режим множественного выбора                            | Выделить несколько фото/песен/задач                    |
| Предпросмотр / Peek & Pop (3D Touch / Haptic Touch) | Показать preview-контроллер                                     | Долгое нажатие на ссылку/контакт/сообщение             |
| Кастомные действия в коллекциях/таблицах            | Показать скрытые кнопки или меню                                | Долгое нажатие на карточку товара → быстрые действия   |

### Что необходимо сделать, чтобы начать использовать UILongPressGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + [[@MainActor]])

```swift
import UIKit

final class TasksViewController: UIViewController {
    
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableView.dataSource = self
        tableView.delegate = self
        
        view.addSubview(tableView)
        tableView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        // 1. Создаём жест долгого нажатия
        let longPress = UILongPressGestureRecognizer(target: self, 
                                                    action: #selector(handleLongPress))
        
        // 2. Настраиваем параметры (очень важно!)
        longPress.minimumPressDuration = 0.5      // минимум 0.5 секунды
        longPress.numberOfTouchesRequired = 1     // одним пальцем
        longPress.allowableMovement = 10          // допустимое смещение пальца (в пунктах)
        
        // 3. Добавляем жест к таблице (или к отдельной ячейке)
        tableView.addGestureRecognizer(longPress)
    }
    
    @objc private func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
        // Жест может быть в состояниях: .began, .changed, .ended, .cancelled
        
        if gesture.state == .began {
            // 4. Получаем точку нажатия относительно таблицы
            let point = gesture.location(in: tableView)
            
            // 5. Находим indexPath ячейки, на которую нажали
            if let indexPath = tableView.indexPathForRow(at: point) {
                let task = tasks[indexPath.row]
                
                // 6. Показываем контекстное меню
                showContextMenu(for: task, at: indexPath)
            }
        }
    }
    
    private func showContextMenu(for task: Task, at indexPath: IndexPath) {
        let alert = UIAlertController(title: task.title, message: nil, preferredStyle: .actionSheet)
        
        alert.addAction(UIAlertAction(title: "Завершить", style: .default) { _ in
            // toggle completion
        })
        
        alert.addAction(UIAlertAction(title: "Удалить", style: .destructive) { _ in
            // delete task
        })
        
        alert.addAction(UIAlertAction(title: "Отмена", style: .cancel))
        
        present(alert, animated: true)
    }
}
```

### Лучшие практики UILongPressGestureRecognizer в Swift 2026

- **minimumPressDuration** — 0.3–0.6 сек (0.5 — стандарт)  
- **allowableMovement** — 10–15 пунктов (чтобы не срабатывало при случайном сдвиге)  
- **location(in:)** — всегда получай точку относительно нужной вью (tableView / collectionView)  
- **indexPathForRow(at:)** — для таблиц/коллекций — самый надёжный способ узнать, на какую ячейку нажали  
- **[[@objc]]** — обязательно для метода-обработчика  
- **[[@MainActor]]** — все обработчики жестов — на главном акторе  
- **[[Swift]] 6 strict concurrency** — [[UIGestureRecognizer]] полностью безопасен  
- **Документируйте** — пиши комментарий «UILongPressGestureRecognizer — контекстное меню при долгом нажатии на задачу»

**Короткий девиз 2026**:
> UILongPressGestureRecognizer — это когда тебе нужно **активировать действие после удержания пальца** (0.5 сек): показать меню, начать drag & drop, включить режим редактирования.  
> В 2026 году это **основной жест** для контекстных действий в списках, галереях и карточках.  
> Всегда проверяй gesture.state == .began и location(in:) для определения цели.
