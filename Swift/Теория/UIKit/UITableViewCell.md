## 📘 Определение

**`UITableViewCell`** — это **класс ячейки таблицы ([[Swift/Теория/UIKit/TableView/UITableView]])** в [[UIKit]], который отвечает за отображение **одной строки** в таблице.  
Позволяет добавлять **текст, изображения, аксессуары и настраиваемые элементы интерфейса**.  
Относится к **UIKit → UITableView / UI Components**.

---

## 🔹 Примеры кода

### 1. Создание стандартной ячейки

```swift
import UIKit

let cell = UITableViewCell(style: .default, reuseIdentifier: "cell")
cell.textLabel?.text = "Привет, UITableViewCell"
```

---

### 2. Использование subtitle стиля

```swift
let cell = UITableViewCell(style: .subtitle, reuseIdentifier: "cell")
cell.textLabel?.text = "Главный текст"
cell.detailTextLabel?.text = "Подпись"
```

---

### 3. Настройка accessoryType

```swift
let cell = UITableViewCell(style: .default, reuseIdentifier: "cell")
cell.accessoryType = .disclosureIndicator // стрелка справа
```

---

### 4. Регистрация ячейки для UITableView

```swift
let tableView = UITableView()
tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
```

---

### 5. Использование в [[UITableViewDataSource]]

```swift
class ViewController: UIViewController, UITableViewDataSource {
    let tableView = UITableView()
    let items = ["One", "Two", "Three"]

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.dataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = items[indexPath.row]
        return cell
    }
}
```

---

### 6. Создание кастомной ячейки

```swift
class CustomCell: UITableViewCell {
    let label = UILabel()
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        contentView.addSubview(label)
        label.frame = contentView.bounds
        label.textAlignment = .center
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
