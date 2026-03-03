#extension #uitableview #uitableviewcell #protocol  

---
# [[UIKit]] — Расширения для [[UITableView]]

Упрощают регистрацию и переиспользование ячеек, хедеров и футеров в таблицах.  
Самые популярные идиоматичные методы в [[Swift]]-проектах 2023–2026 годов.

```swift
extension UITableView {
    /// Регистрирует несколько типов ячеек одновременно
    func registerCells(_ cells: UITableViewCell.Type...) {
        cells.forEach { cellType in
            let identifier = String(describing: cellType)
            register(cellType, forCellReuseIdentifier: identifier)
        }
    }
    
    /// Получает переиспользуемую ячейку с приведением к нужному типу
    func reuseCell<T: UITableViewCell>(_ type: T.Type, for indexPath: IndexPath) -> T {
        let identifier = String(describing: type)
        return dequeueReusableCell(withIdentifier: identifier, for: indexPath) as! T
    }
    
    /// Регистрирует несколько типов хедеров
    func registerHeaders(_ headerTypes: UITableViewHeaderFooterView.Type...) {
        headerTypes.forEach { headerType in
            let identifier = String(describing: headerType)
            register(headerType, forHeaderFooterViewReuseIdentifier: identifier)
        }
    }
    
    /// Регистрирует несколько типов футеров
    func registerFooters(_ footerTypes: UITableViewHeaderFooterView.Type...) {
        footerTypes.forEach { footerType in
            let identifier = String(describing: footerType)
            register(footerType, forHeaderFooterViewReuseIdentifier: identifier)
        }
    }
    
    /// Получает переиспользуемый хедер с приведением к типу
    func reuseHeader<T: UITableViewHeaderFooterView>(_ type: T.Type) -> T? {
        let identifier = String(describing: type)
        return dequeueReusableHeaderFooterView(withIdentifier: identifier) as? T
    }
    
    /// Получает переиспользуемый футер с приведением к типу
    func reuseFooter<T: UITableViewHeaderFooterView>(_ type: T.Type) -> T? {
        let identifier = String(describing: type)
        return dequeueReusableHeaderFooterView(withIdentifier: identifier) as? T
    }
}
```

## Зачем нужны эти методы?

1. **Убирают дублирование строк с `String(describing:)`**  
   Вместо:

   ```swift
   tableView.register(ProfileCell.self, forCellReuseIdentifier: "ProfileCell")
   tableView.register(NewsCell.self,   forCellReuseIdentifier: "NewsCell")
   ```

   Пишем одну строку:

   ```swift
   tableView.registerCells(ProfileCell.self, NewsCell.self)
   ```

2. **Безопасное приведение типов без `as!`**  
   `reuseCell` возвращает уже приведённый тип → нет риска краша из-за неправильного каста.

3. **Симметрия для ячеек / хедеров / футеров**  
   Один и тот же паттерн для всех частей таблицы.

4. **Меньше опечаток**  
   Идентификатор всегда генерируется автоматически из имени класса.

## Типичные сценарии использования

### Вариант 1 — регистрация в [[viewDidLoad]] / [[awakeFromNib]]

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tableView.registerCells(
        UserCell.self,
        PostCell.self,
        LoadingCell.self
    )
    
    tableView.registerHeaders(SectionHeaderView.self)
    tableView.registerFooters(SectionFooterView.self)
}
```

### Вариант 2 — в `cellForRow` и `viewForHeaderInSection`

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    switch sectionType {
    case .user:
        let cell = tableView.reuseCell(UserCell.self, for: indexPath)
        cell.configure(with: user)
        return cell
        
    case .post:
        return tableView.reuseCell(PostCell.self, for: indexPath)
            .configure(with: posts[indexPath.row])
    }
}

func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    tableView.reuseHeader(SectionHeaderView.self)?
        .configure(title: sectionTitles[section])
}
```

## Рекомендуемые улучшения (часто добавляют)

```swift
// Более безопасная версия без force unwrap (если боитесь крашей)
func dequeue<T: UITableViewCell>(_ type: T.Type, for indexPath: IndexPath) -> T {
    let id = String(describing: type)
    guard let cell = dequeueReusableCell(withIdentifier: id, for: indexPath) as? T else {
        fatalError("Не удалось dequeue \(id). Зарегистрирована ли ячейка?")
    }
    return cell
}

// Или ещё короче (самый популярный вариант 2025+)
func cell<T: UITableViewCell>(ofType: T.Type, at indexPath: IndexPath) -> T {
    dequeueReusableCell(withIdentifier: String(describing: ofType), for: indexPath) as! T
}
```

## Сравнение старого и нового стиля

| Задача                              | Старый стиль (классика)                                      | Новый стиль (с extension)                          |
|-------------------------------------|---------------------------------------------------------------|----------------------------------------------------|
| Регистрация ячеек                   | 3–5 строк register                                        | 1 строка `registerCells(...)`                      |
| Получение ячейки                    | `as! UserCell` или `as?` + guard                          | `reuseCell(UserCell.self, for: indexPath)`         |
| Регистрация хедера                  | отдельный `register(..., forHeaderFooterViewReuseIdentifier:)` | `registerHeaders(Header.self)`                     |
| Получение хедера                    | `dequeueReusableHeaderFooterView` + `as?`                 | `reuseHeader(Header.self)`                         |

## Полный пример ViewController’а

```swift
class FeedViewController: UIViewController, UITableViewDataSource {
    
    @IBOutlet private weak var tableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.dataSource = self
        
        tableView.registerCells(
            PostCell.self,
            StoryCell.self,
            AdCell.self
        )
        
        tableView.registerHeaders(FeedSectionHeader.self)
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        posts.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let post = posts[indexPath.row]
        
        if post.isSponsored {
            return tableView.reuseCell(AdCell.self, for: indexPath)
                .configure(with: post)
        } else {
            return tableView.reuseCell(PostCell.self, for: indexPath)
                .configure(with: post)
        }
    }
    
    func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
        tableView.reuseHeader(FeedSectionHeader.self)?
            .setTitle("Сегодня")
    }
}
```

---
