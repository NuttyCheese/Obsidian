`UITableViewDataSource` - это протокол в [[iOS]] SDK, который определяет методы, используемые для предоставления данных и управления содержимым таблицы ([[Swift/Теория/Swift/UIKit/TableView/UITableView]]). Класс, который реализует этот протокол, действует в качестве источника данных для таблицы и предоставляет информацию о количестве секций, количестве строк в каждой секции и содержимом каждой ячейки.

Основные методы `UITableViewDataSource`:

1. **numberOfSections(in:)**: Этот метод возвращает количество секций в таблице. Если вы не используете секции, можете вернуть 1.
    
2. **tableView(_:numberOfRowsInSection:)**: Этот метод возвращает количество строк в указанной секции таблицы.
    
3. **tableView(_:cellForRowAt:)**: Этот метод возвращает ячейку для конкретной строки таблицы. Обычно в этом методе вы создаете или переиспользуете ячейку, заполняете ее данными и возвращаете.
    
4. **tableView(_:titleForHeaderInSection:) (необязательный)**: Этот метод возвращает заголовок для указанной секции таблицы. Он используется, когда в вашей таблице есть секции.
    
5. **tableView(_:titleForFooterInSection:) (необязательный)**: Этот метод возвращает текст для подвала указанной секции таблицы.
    

Пример использования `UITableViewDataSource`:
```swift
import UIKit

class MyTableViewController: UIViewController, UITableViewDataSource {
    let data = ["Item 1", "Item 2", "Item 3"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let tableView = UITableView(frame: view.bounds, style: .plain)
        tableView.dataSource = self
        view.addSubview(tableView)
    }
    
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell(style: .default, reuseIdentifier: nil)
        cell.textLabel?.text = data[indexPath.row]
        return cell
    }
}

```
В этом примере создается простой контроллер таблицы, который реализует методы `UITableViewDataSource`. Он показывает одну секцию с несколькими строками данных из массива [[Data]]. Каждая строка представлена стандартной ячейкой с текстом из массива.