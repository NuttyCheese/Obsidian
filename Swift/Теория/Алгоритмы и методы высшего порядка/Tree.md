#swift #data_structures #tree #hierarchy #nodes
# Деревья (Tree) в [[Swift]]

**Дерево** — иерархическая структура данных, состоящая из **узлов** (nodes), связанных **рёбрами** (edges). У каждого узла (кроме корня) есть **ровно один родитель**, а у узла может быть **произвольное количество детей**.

### Основные свойства деревьев

- Один **корень** (root)
- Нет циклов
- Каждый узел, кроме корня, имеет **ровно одного родителя**
- Высота дерева — длина самого длинного пути от корня до листа
- Лист (leaf) — узел без детей
- Поддерево — дерево, состоящее из узла и всех его потомков

### Виды деревьев, часто встречающихся в iOS

- **Общее дерево** (n-ary tree) — любой узел может иметь любое количество детей
- **Бинарное дерево** (binary tree) — не более двух детей
- **Бинарное дерево поиска** (BST)
- **AVL-дерево**, **красно-чёрное дерево** (используются внутри [[Dictionary]] / [[Set]])
- **Дерево решений**, **четырёхугольное дерево** (Quadtree), **октодерево** (Octree) — в играх и AR
- **DOM-дерево** (в WebKit), **иерархия [[Swift/Расширения/UIView]]**, **файловая система**

## 1. Базовая структура общего дерева

```swift
class TreeNode<T> {
    var value: T
    var children: [TreeNode<T>] = []
    weak var parent: TreeNode<T>?   // полезно для обратной связи
    
    init(value: T) {
        self.value = value
    }
    
    func add(child: TreeNode<T>) {
        children.append(child)
        child.parent = self
    }
    
    func remove(child: TreeNode<T>) {
        children.removeAll { $0 === child }
        child.parent = nil
    }
}
```

## 2. Создание и визуализация дерева

```swift
let root = TreeNode(value: "Документы")
let photos = TreeNode(value: "Фото")
let work = TreeNode(value: "Работа")
let vacation = TreeNode(value: "Отпуск 2025")

root.add(child: photos)
root.add(child: work)

photos.add(child: TreeNode(value: "iPhone"))
photos.add(child: TreeNode(value: "Камера"))

work.add(child: vacation)
vacation.add(child: TreeNode(value: "Италия"))
vacation.add(child: TreeNode(value: "Япония"))

// Рекурсивный вывод с отступами
func printTree<T>(_ node: TreeNode<T>, prefix: String = "", isLast: Bool = true) {
    let line = isLast ? "└── " : "├── "
    print(prefix + line + "\(node.value)")
    
    let newPrefix = prefix + (isLast ? "    " : "│   ")
    for (index, child) in node.children.enumerated() {
        printTree(child, prefix: newPrefix, isLast: index == node.children.count - 1)
    }
}

printTree(root)
/*
Документы
├── Фото
│   ├── iPhone
│   └── Камера
└── Работа
    └── Отпуск 2025
        ├── Италия
        └── Япония
*/
```

## 3. Обходы дерева (Traversal)

### DFS — поиск в глубину (Depth-First Search)

```swift
// Рекурсивный pre-order (сначала корень)
func dfsPreOrder<T>(_ node: TreeNode<T>, visit: (T) -> Void) {
    visit(node.value)
    node.children.forEach { dfsPreOrder($0, visit: visit) }
}

// In-order (для бинарных деревьев имеет смысл, здесь — просто обход)
func dfsInOrder<T>(_ node: TreeNode<T>, visit: (T) -> Void) {
    if node.children.count == 2 {
        dfsInOrder(node.children[0], visit: visit)
        visit(node.value)
        dfsInOrder(node.children[1], visit: visit)
    } else {
        visit(node.value)
        node.children.forEach { dfsInOrder($0, visit: visit) }
    }
}

// Post-order (сначала дети, потом корень)
func dfsPostOrder<T>(_ node: TreeNode<T>, visit: (T) -> Void) {
    node.children.forEach { dfsPostOrder($0, visit: visit) }
    visit(node.value)
}
```

### BFS — поиск в ширину (Breadth-First Search / Level Order)

```swift
func bfsLevelOrder<T>(_ root: TreeNode<T>, visit: (T) -> Void) {
    guard root.children.count > 0 || true else { return }
    
    var queue: [TreeNode<T>] = [root]
    
    while !queue.isEmpty {
        let node = queue.removeFirst()
        visit(node.value)
        
        queue.append(contentsOf: node.children)
    }
}

// Вывод по уровням
func printLevels<T>(_ root: TreeNode<T>) {
    var queue: [(node: TreeNode<T>, level: Int)] = [(root, 0)]
    var lastLevel = -1
    
    while !queue.isEmpty {
        let (node, level) = queue.removeFirst()
        
        if level > lastLevel {
            print("\nУровень \(level): ", terminator: "")
            lastLevel = level
        }
        
        print("\(node.value) ", terminator: "")
        
        for child in node.children {
            queue.append((child, level + 1))
        }
    }
    print()
}
```

## 4. Полезные операции над деревом

```swift
extension TreeNode {
    // Подсчёт всех узлов
    var nodeCount: Int {
        1 + children.reduce(0) { $0 + $1.nodeCount }
    }
    
    // Глубина / высота поддерева
    var height: Int {
        children.isEmpty ? 0 : 1 + children.map(\.height).max()!
    }
    
    // Все листья
    var leaves: [TreeNode<T>] {
        children.isEmpty ? [self] : children.flatMap { $0.leaves }
    }
    
    // Поиск по значению (DFS)
    func find(_ value: T) -> TreeNode<T>? where T: Equatable {
        if self.value == value { return self }
        for child in children {
            if let found = child.find(value) { return found }
        }
        return nil
    }
    
    // Уровень узла от корня
    var levelFromRoot: Int {
        var level = 0
        var current = self
        while let parent = current.parent {
            level += 1
            current = parent
        }
        return level
    }
}
```

## 5. Реальные сценарии в [[iOS]]

### Пример 1 — иерархия категорий товаров

```swift
let rootCategory = TreeNode(value: "Все товары")

let electronics = TreeNode(value: "Электроника")
let clothing    = TreeNode(value: "Одежда")

rootCategory.add(child: electronics)
rootCategory.add(child: clothing)

electronics.add(child: TreeNode(value: "Смартфоны"))
electronics.add(child: TreeNode(value: "Ноутбуки"))

clothing.add(child: TreeNode(value: "Мужская"))
clothing.add(child: TreeNode(value: "Женская"))

// Отображаем в UITableView с отступами
func configureCell(for node: TreeNode<String>, at indexPath: IndexPath) {
    let level = node.levelFromRoot
    cell.textLabel?.text = String(repeating: "  ", count: level) + node.value
}
```

### Пример 2 — дерево комментариев (как в соцсетях)

```swift
class CommentNode: TreeNode<String> {
    let author: String
    let text: String
    
    init(author: String, text: String) {
        self.author = author
        self.text = text
        super.init(value: "\(author): \(text.prefix(30))…")
    }
}

// Построение дерева комментариев
let post = CommentNode(author: "user1", text: "Отличное фото!")
let reply1 = CommentNode(author: "user2", text: "Согласен!")
let reply2 = CommentNode(author: "user3", text: "Класс!")

post.add(child: reply1)
post.add(child: reply2)

reply1.add(child: CommentNode(author: "user1", text: "Спасибо!"))
```

### Пример 3 — обход дерева UI-компонентов ([[Swift/Расширения/UIView]] hierarchy)

```swift
extension UIView {
    var treeDescription: String {
        var result = "\(type(of: self))"
        if let label = (self as? UILabel)?.text {
            result += " \"\(label)\""
        }
        return result
    }
    
    func printViewHierarchy(level: Int = 0) {
        print(String(repeating: "  ", count: level) + treeDescription)
        for subview in subviews {
            subview.printViewHierarchy(level: level + 1)
        }
    }
}

// Использование
view.printViewHierarchy()
```

## Итог — когда использовать деревья в iOS

| Задача                              | Тип дерева / структура               | Алгоритм обхода |
|-------------------------------------|--------------------------------------|------------------|
| Файловая система                    | Общее дерево                         | DFS / BFS        |
| Иерархия UIView / UI компонентов    | Общее дерево                         | DFS (чаще всего) |
| Комментарии / ветки обсуждений      | Общее дерево                         | DFS / BFS по уровням |
| Категории товаров / меню            | Общее дерево                         | BFS (для отображения уровней) |
| Поиск по значению                   | Общее дерево                         | DFS              |
| Кэш LRU / приоритетная очередь      | —                                    | —                |
| Дерево решений / DOM                | Общее дерево                         | DFS              |

**Короткое правило**:
> В iOS-разработке **деревья** встречаются везде: от иерархии view до структуры комментариев и категорий.  
> Для большинства задач достаточно **рекурсивного DFS** и **BFS по уровням**.  
> Для сложных случаев (поиск, балансировка) лучше использовать готовые структуры или сторонние библиотеки (Swift Collections, Graph).
