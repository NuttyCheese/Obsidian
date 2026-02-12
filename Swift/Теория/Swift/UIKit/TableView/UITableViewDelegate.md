`UITableViewDelegate` - это протокол в [[iOS]] SDK, который определяет методы для настройки и управления поведением таблицы ([[Swift/Теория/Swift/UIKit/TableView/UITableView]]). Объект, который реализует этот протокол, действует в качестве делегата для таблицы и может обрабатывать события, такие как выбор строки, изменение высоты строк, заголовков и другие.

Некоторые основные методы `UITableViewDelegate`:

1. **tableView(_:didSelectRowAt:)**: Этот метод вызывается при выборе строки пользователем. Он позволяет реагировать на выбор определенной строки и выполнять соответствующие действия.
    
2. **tableView(_:heightForRowAt:)**: Этот метод возвращает высоту строки для указанной строки таблицы. Он используется, если вы хотите, чтобы строки имели разную высоту.
    
3. **tableView(_:viewForHeaderInSection:)**: Этот метод возвращает представление (view) для заголовка указанной секции. Он используется, когда в таблице есть секции и требуется настроить внешний вид заголовка.
    
4. **tableView(_:didEndDisplaying:forRowAt:)**: Этот метод вызывается после того, как строка перестала отображаться на экране. Он может быть использован для выполнения дополнительных действий, например, когда нужно освободить ресурсы строки.
    
5. **tableView(_:willDisplayHeaderView:forSection:)**: Этот метод вызывается перед отображением заголовка секции на экране. Он позволяет настроить внешний вид заголовка перед его отображением.
    

Пример использования `UITableViewDelegate`:
```swift
import UIKit

class MyTableViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
    let data = ["Item 1", "Item 2", "Item 3"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let tableView = UITableView(frame: view.bounds, style: .plain)
        tableView.dataSource = self
        tableView.delegate = self // Установка делегата
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
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("Выбрана строка с индексом \(indexPath.row)")
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 50 // Возвращаем высоту строки
    }
}

```
В этом примере используется `UITableViewDelegate` для обработки событий выбора строки (`didSelectRowAt`) и настройки высоты строк таблицы (`heightForRowAt`). Делегат также используется для настройки и управления другими аспектами поведения таблицы в зависимости от потребностей приложения.