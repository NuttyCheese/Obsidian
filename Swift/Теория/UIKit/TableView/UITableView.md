**UITableView** — это один из самых популярных и часто используемых компонентов в [[UIKit]] для отображения **списков данных** (одна колонка, вертикальная прокрутка).

В 2026 году он остаётся **основным инструментом** для всех классических списков в [[iOS]]-приложениях: настройки, чаты, контакты, новости, история заказов, комментарии и т.д.

### Для чего использовать UITableView (реальные сценарии 2026)

| Сценарий                                         | Почему именно UITableView                            | Альтернатива (если не подходит)                    |
| ------------------------------------------------ | ---------------------------------------------------- | -------------------------------------------------- |
| Настройки приложения (Settings-like)             | Нативный стиль, группировка, [[switch]], disclosure  | [[SwiftUI]] List (если чистый SwiftUI)             |
| Чат (сообщения)                                  | Динамическая высота ячеек, автоматическая прокрутка  | SwiftUI List + LazyVStack                          |
| Список контактов / поиск / фильтры               | Поддержка index titles, searchController             | SwiftUI List + searchable                          |
| История заказов, транзакций, уведомлений         | Простота, высокая производительность на тысячи строк | SwiftUI List                                       |
| Таблица с группировкой (разделы по буквам/датам) | Встроенная поддержка секций + заголовки              | SwiftUI List + Section                             |
| Простой список с swipe actions / edit mode       | Нативные swipe, delete, move                         | SwiftUI List + .onDelete                           |
| Экран с очень большим количеством строк          | Лучшая оптимизация под тысячи элементов              | SwiftUI LazyVStack (хуже на очень больших списках) |

### Что необходимо сделать, чтобы начать использовать UITableView

Вот **минимальный, но современный** чек-лист на 2026 год (программный подход, без Storyboard).

#### 1. Создать UITableView

```swift
let tableView = UITableView(frame: .zero, style: .insetGrouped) // или .plain, .grouped
tableView.backgroundColor = .systemBackground
tableView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(tableView)
```

#### 2. Зарегистрировать ячейки (обязательно!)

```swift
// Вариант 1 — кастомная ячейка из кода
tableView.register(CustomCell.self, forCellReuseIdentifier: "CustomCell")

// Вариант 2 — из xib
tableView.register(UINib(nibName: "CustomCell", bundle: nil), 
                  forCellReuseIdentifier: "CustomCell")

// Вариант 3 — стандартная ячейка
tableView.register(UITableViewCell.self, forCellReuseIdentifier: "DefaultCell")
```

#### 3. Назначить dataSource и [[delegate]] (обычно [[self]])

```swift
tableView.dataSource = self
tableView.delegate = self
```

#### 4. Реализовать минимум два метода DataSource

```swift
extension YourViewController: UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "CustomCell", for: indexPath) as! CustomCell
        let item = items[indexPath.row]
        cell.configure(with: item)
        return cell
    }
}
```

#### 5. (Опционально) Добавить делегат для кликов и других событий

```swift
extension YourViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let item = items[indexPath.row]
        // открыть детальный экран
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 80 // или UITableView.automaticDimension для динамической высоты
    }
}
```

#### 6. (Важно!) Обновлять таблицу при изменении данных

```swift
// Самый простой способ
tableView.reloadData()

// Лучше — точечное обновление (анимации + производительность)
tableView.performBatchUpdates({
    tableView.insertRows(at: [newIndexPath], with: .automatic)
    tableView.deleteRows(at: [oldIndexPath], with: .automatic)
}, completion: nil)

// Самый современный способ 2026 — UITableViewDiffableDataSource
dataSource.apply(snapshot, animatingDifferences: true)
```

### Короткий чек-лист «Что нужно сделать, чтобы UITableView заработал»

1. Создать UITableView (style: .plain / .grouped / .insetGrouped)  
2. Зарегистрировать все ячейки (обязательно!)  
3. Назначить dataSource и delegate  
4. Реализовать минимум `numberOfRowsInSection` и `cellForRowAt`  
5. Вызвать `reloadData()` после изменения данных  
6. Добавить constraints или frame  
7. (Опционально) Реализовать delegate для кликов, высоты, swipe actions

### Короткий девиз 2026

> UITableView — это когда тебе нужен **простой, быстрый, вертикальный список** с нативным стилем iOS (настройки, чаты, контакты, история).  
> В 2026 году это **основной инструмент** для классических списков в UIKit.  
> Если нужна сетка, горизонтальный скролл или разные размеры — сразу бери **UICollectionView**.
