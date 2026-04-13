#foundation #indexpath #uitableview #uicollectionview #ios #swift

---

## IndexPath — Путь к элементу в иерархических коллекциях

### Определение

**`IndexPath`** — это структура в [[Foundation]], которая представляет **путь к элементу** в иерархической коллекции, такой как [[UITableView]] или [[UICollectionView]]. Каждый `IndexPath` содержит массив целых чисел ([[Int]]), где каждый элемент массива указывает на индекс на соответствующем уровне вложенности.

В контексте iOS-разработки `IndexPath` чаще всего используется для идентификации конкретной ячейки в таблице или коллекции, указывая её **секцию** и **строку** (для `UITableView`) или **секцию** и **элемент** (для `UICollectionView`).

### Зачем это знать iOS-разработчику?

1.  **Идентификация ячеек:** `IndexPath` используется в методах [[UITableViewDataSource]] и [[UICollectionViewDataSource]] (`cellForRowAt`, `numberOfRowsInSection`).
2.  **Обработка выбора:** Методы делегатов (`didSelectRowAt`, `didSelectItemAt`) передают `IndexPath` выбранной ячейки.
3.  **Вставка и удаление:** Методы `insertRows(at:)`, `deleteRows(at:)`, `reloadRows(at:)` используют массивы `IndexPath`.
4.  **Перемещение:** Метод `moveRow(at:to:)` использует `IndexPath` для источника и назначения.
5.  **Секции и элементы:** Удобный доступ к номерам секций (`section`) и строк/элементов (`row`/`item`).

---

### История и эволюция

| Версия             | Изменение                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| **До [[Swift]] 3** | `IndexPath` был классом, унаследованным от `NSIndexPath`.                                           |
| **Swift 3+**       | `IndexPath` стал структурой ([[value type]]) с типизированными свойствами `section`, `row`, `item`. |
| **[[iOS]] 13+**    | Добавлена поддержка `Item` для `UICollectionView` (аналогично `row`).                               |

---

### Создание IndexPath

#### 1. **С помощью инициализатора (рекомендуемый способ)**

```swift
import Foundation

// Для UITableView (секция, строка)
let indexPath = IndexPath(row: 0, section: 1)

// Для UICollectionView (секция, элемент)
let indexPath = IndexPath(item: 5, section: 2)
```

#### 2. **С помощью массива индексов**

```swift
let indexPath = IndexPath(indexes: [1, 0])  // section = 1, row/item = 0
```

#### 3. **Преобразование из NSIndexPath (в редких случаях)**

```swift
let nsIndexPath = NSIndexPath(row: 2, section: 1)
let indexPath = nsIndexPath as IndexPath
```

---

### Свойства IndexPath

| Свойство      | Тип      | Описание                                 |
| ------------- | -------- | ---------------------------------------- |
| **`section`** | `Int`    | Номер секции (первый индекс в пути).     |
| **`row`**     | `Int`    | Номер строки (для `UITableView`).        |
| **`item`**    | `Int`    | Номер элемента (для `UICollectionView`). |
| **`count`**   | [[Int]]  | Количество элементов в пути (обычно 2).  |
| **`isEmpty`** | [[Bool]] | Пустой ли путь.                          |
| **`indices`** | `[Int]`  | Массив всех индексов в пути.             |

```swift
let indexPath = IndexPath(row: 3, section: 2)
print(indexPath.section)   // 2
print(indexPath.row)       // 3
print(indexPath.item)      // 3 (для UICollectionView)
print(indexPath.count)     // 2
print(indexPath.isEmpty)   // false
```

---

### Работа с секциями и элементами

#### 1. **Создание и доступ**

```swift
var indexPath = IndexPath(row: 0, section: 0)
indexPath.row = 5
indexPath.section = 2
print(indexPath)  // {length = 2, path = 2 - 5}
```

#### 2. **Сравнение IndexPath**

```swift
let path1 = IndexPath(row: 0, section: 0)
let path2 = IndexPath(row: 0, section: 0)
let path3 = IndexPath(row: 1, section: 0)

path1 == path2  // true
path1 == path3  // false
```

#### 3. **Сортировка массивов IndexPath**

```swift
var paths = [
    IndexPath(row: 2, section: 1),
    IndexPath(row: 0, section: 0),
    IndexPath(row: 1, section: 1),
    IndexPath(row: 0, section: 0)
]

paths.sort()
print(paths)
// [0-0, 0-0, 1-1, 2-1] (сначала по section, потом по row)
```

---

### Использование в UITableView

#### 1. **DataSource методы**

```swift
import UIKit

class TableViewController: UITableViewController {
    let data = [
        ["Apple", "Banana", "Orange"],
        ["Car", "Bike", "Bus"],
        ["Red", "Green", "Blue"]
    ]
    
    override func numberOfSections(in tableView: UITableView) -> Int {
        return data.count
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data[section].count
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        cell.textLabel?.text = data[indexPath.section][indexPath.row]
        return cell
    }
}
```

#### 2. **Обработка выбора**

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let selectedItem = data[indexPath.section][indexPath.row]
    print("Выбрано: \(selectedItem)")
    print("Секция: \(indexPath.section), строка: \(indexPath.row)")
}
```

#### 3. **Редактирование (вставка, удаление)**

```swift
// Добавление новой строки
let newIndexPath = IndexPath(row: data[0].count, section: 0)
data[0].append("Mango")
tableView.insertRows(at: [newIndexPath], with: .automatic)

// Удаление строки
let deleteIndexPath = IndexPath(row: 0, section: 0)
data[0].remove(at: 0)
tableView.deleteRows(at: [deleteIndexPath], with: .fade)
```

#### 4. **Перемещение ячеек**

```swift
override func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
    let movedItem = data[sourceIndexPath.section][sourceIndexPath.row]
    data[sourceIndexPath.section].remove(at: sourceIndexPath.row)
    data[destinationIndexPath.section].insert(movedItem, at: destinationIndexPath.row)
}
```

---

### Использование в UICollectionView

```swift
class CollectionViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate {
    @IBOutlet weak var collectionView: UICollectionView!
    
    let items = Array(1...100)
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return items.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath)
        // Здесь item используется вместо row
        let value = items[indexPath.item]
        // Настройка ячейки
        return cell
    }
    
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        print("Выбран элемент: \(items[indexPath.item])")
    }
}
```

---

### Преобразование между row и item

```swift
let tableViewIndexPath = IndexPath(row: 5, section: 2)
let collectionViewIndexPath = IndexPath(item: tableViewIndexPath.row, section: tableViewIndexPath.section)

// Внимание: для UICollectionView принято использовать item, но row тоже работает
let samePath = IndexPath(row: 3, section: 1)  // item тоже будет 3
```

---

### IndexPath и множественные изменения

```swift
// Массовое удаление
var indexPathsToDelete: [IndexPath] = []
for section in 0..<data.count {
    for row in 0..<data[section].count {
        if data[section][row].hasPrefix("A") {
            indexPathsToDelete.append(IndexPath(row: row, section: section))
        }
    }
}
// Удаляем с анимацией
tableView.deleteRows(at: indexPathsToDelete, with: .automatic)
```

---

### IndexPath в diffable data source

```swift
import UIKit

class DiffableTableViewController: UITableViewController {
    enum Section {
        case main
    }
    
    var dataSource: UITableViewDiffableDataSource<Section, String>!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        dataSource = UITableViewDiffableDataSource<Section, String>(tableView: tableView) { tableView, indexPath, itemIdentifier in
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
            cell.textLabel?.text = itemIdentifier
            return cell
        }
        
        var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
        snapshot.appendSections([.main])
        snapshot.appendItems(["Apple", "Banana", "Orange"])
        dataSource.apply(snapshot, animatingDifferences: true)
    }
    
    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        guard let item = dataSource.itemIdentifier(for: indexPath) else { return }
        print("Выбрано: \(item)")
    }
}
```

---

### IndexPath вне UI (общие иерархии)

```swift
struct TreeNode {
    let value: String
    var children: [TreeNode] = []
}

func findNode(at indexPath: IndexPath, in root: TreeNode) -> TreeNode? {
    var currentNode = root
    for index in indexPath.indices {
        guard index < currentNode.children.count else { return nil }
        currentNode = currentNode.children[index]
    }
    return currentNode
}

let tree = TreeNode(value: "Root", children: [
    TreeNode(value: "Child 1"),
    TreeNode(value: "Child 2", children: [
        TreeNode(value: "Grandchild")
    ])
])

let indexPath = IndexPath(indexes: [1, 0])
let node = findNode(at: indexPath, in: tree)
print(node?.value ?? "nil")  // Grandchild
```

---

### Лучшие практики

1.  **Используйте `row` для `UITableView`, `item` для `UICollectionView`** — код будет понятнее.
2.  **Никогда не храните `IndexPath` дольше, чем нужно** — они могут стать неактуальными после изменений в данных.
3.  **При вставке/удалении используйте `performBatchUpdates`** для анимации и консистентности.
4.  **Для Diffable Data Source не храните `IndexPath`** — используйте `itemIdentifier`.

```swift
// Правильно: получаем свежий IndexPath при необходимости
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    // используем indexPath здесь
}

// Неправильно: сохраняем IndexPath на будущее
var selectedIndexPath: IndexPath?  // может стать невалидным
```

---

### Итог

**`IndexPath`** — это фундаментальная структура для работы с иерархическими коллекциями в iOS. Она позволяет:

1.  **Идентифицировать ячейки, секции и элементы** в таблицах и коллекциях.
2.  **Выполнять вставку, удаление, перемещение** с анимацией.
3.  **Обрабатывать пользовательский выбор** и передавать данные.
4.  **Работать с иерархическими данными** в общем случае.
5.  **Интегрироваться с современными Diffable Data Source**.

Понимание `IndexPath` необходимо для эффективной работы с `UITableView`, `UICollectionView` и любыми иерархическими структурами данных .