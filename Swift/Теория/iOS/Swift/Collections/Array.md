**Array** в [[Swift]] — это **упорядоченная коллекция элементов одного типа**, которая хранит значения последовательно и позволяет обращаться к ним по индексу (начиная с 0).

Это один из самых часто используемых типов в Swift — почти в каждом приложении есть десятки массивов: списки пользователей, товары в корзине, сообщения в чате, координаты маршрута, результаты поиска и т.д.

### 1. Основные свойства и особенности Array (2026 актуально)

| Свойство / Поведение            | Описание                                                               | Важные детали 2026                                       |
| ------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------- |
| **Тип элементов**               | Все элементы одного типа `Element`                                     | `Array<Int>`, `Array<String>`, `Array<UIView>` и т.д.    |
| **[[Value Type]]**              | Копируется при присваивании или передаче                               | `var a = [1,2,3]`<br>`var b = a` → b — независимая копия |
| **[[Mutable]] / [[Immutable]]** | `var` — можно менять, `let` — только читать                            | `let names = ["Anna"]`<br>`names.append("Bob")` → ошибка |
| **Индексация**                  | От 0 до `count-1`                                                      | `array[0]`, `array[array.count-1]`                       |
| **Пустой массив**               | `[]` или `Array<Element>()`                                            | `var tasks: [String] = []`                               |
| **Динамический размер**         | Автоматически растёт/сжимается при append/remove                       | Нет фиксированного размера как в C                       |
| **[[Copy-on-write]] (COW)**     | Копирование происходит только при изменении                            | Очень эффективно в Swift 6                               |
| **Thread-safety**               | Не является thread-safe — требует [[@MainActor]] или [[DispatchQueue]] | В [[Swift]] 6 строже проверяется                         |

### 2. Создание и базовые операции (самые частые в 2026)

```swift
// 1. Пустой массив
var scores: [Int] = []
var names = [String]()           // то же самое

// 2. С начальными значениями
let colors = ["red", "green", "blue"]
var numbers = [1, 2, 3, 4, 5]

// 3. Явный тип (полезно при пустом массиве)
var upcomingEvents: [Event] = []

// 4. Доступ и изменение
print(colors[0])                 // "red"
numbers[2] = 10                  // [1, 2, 10, 4, 5]

// 5. Добавление элементов
scores.append(95)                // добавляет в конец
scores += [88, 92]               // добавляет несколько
scores.insert(100, at: 0)        // вставляет в начало

// 6. Удаление
scores.remove(at: 1)             // удаляет по индексу
scores.removeLast()              // удаляет последний
scores.removeAll()               // очищает массив
```

### 3. Самые полезные методы и свойства (топ-2026)

| Метод / Свойство                               | Что возвращает / делает                    | Пример использования 2026                                  |
| ---------------------------------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| `count` / `isEmpty`                            | Количество элементов / пустой ли массив    | `if users.isEmpty { showEmptyState() }`                    |
| `append(_:)` / `append(contentsOf:)`           | Добавляет один элемент или коллекцию       | `cart.append(newProduct)`                                  |
| `insert(_:at:)`                                | Вставляет элемент по индексу               | `tasks.insert(urgentTask, at: 0)`                          |
| `remove(at:)` / `removeLast()` / `removeAll()` | Удаляет по индексу / последний / всё       | `cart.remove(at: index)`                                   |
| `first` / `last`                               | Первый / последний элемент (опционально)   | `let latest = messages.last`                               |
| `map`, `filter`, `reduce`                      | Высокоуровневые функции (самые популярные) | `let names = users.map { $0.name }`                        |
| `sorted`, `sorted(by:)`                        | Возвращает отсортированную копию           | `tasks.sorted { $0.priority > $1.priority }`               |
| `contains`, `contains(where:)`                 | Проверяет наличие элемента / условия       | `if tasks.contains(where: { $0.isUrgent }) { ... }`        |
| [[compactMap]]                                 | [[map]] + фильтрация [[nil]]               | `let ages = strings.compactMap { Int($0) }`                |
| `joined(separator:)`                           | Объединяет элементы в строку               | `let csv = values.map(String.init).joined(separator: ",")` |

### 4. Полный реальный пример 2026 года (современный стиль)

```swift
struct Task {
    let title: String
    let priority: Int
    let isCompleted: Bool
}

class TaskManager {
    private var tasks: [Task] = []
    
    func addTask(_ title: String, priority: Int = 0) {
        tasks.append(Task(title: title, priority: priority, isCompleted: false))
    }
    
    func completeTask(at index: Int) {
        guard tasks.indices.contains(index) else { return }
        tasks[index].isCompleted = true
    }
    
    var sortedTasks: [Task] {
        tasks.sorted { $0.priority > $1.priority }  // по убыванию приоритета
    }
    
    var pendingTasks: [Task] {
        tasks.filter { !$0.isCompleted }
    }
    
    func search(_ query: String) -> [Task] {
        tasks.filter { $0.title.localizedCaseInsensitiveContains(query) }
    }
    
    func exportToCSV() -> String {
        let headers = "Title,Priority,Completed"
        let rows = tasks.map { "\($0.title),\($0.priority),\($0.isCompleted)" }
        return ([headers] + rows).joined(separator: "\n")
    }
}
```

### 5. Лучшие практики Array в Swift 2026

- **Используй `let`, когда массив не меняется** — это даёт больше оптимизаций компилятору  
- **Предпочитай `Array` над `NSMutableArray`** — `Array` быстрее и безопаснее  
- **Для больших массивов** — используй `reserveCapacity(_:)` перед массовым append  
- **Избегай [[force unwrap]]** — используй `first`, `last`, `safe subscript`  
- **Для многопоточности** — массивы **не thread-safe** — используй `DispatchQueue`, `actor` или `@MainActor`  
- **Swift 6 strict concurrency** — массивы полностью безопасны, но операции изменения — на одном акторе  
- **Документируйте** — пиши комментарий «[Task] — список задач, отсортированный по приоритету»

**Короткий девиз 2026**:
> Array — это **упорядоченный, копируемый, динамический список** элементов одного типа.  
> В 2026 году используй `let` для неизменяемых массивов, [[map]]/[filter]/[[reduce]] для обработки, `UIAction`/[[@MainActor]] для UI, и помни: **копируется только при изменении** (Copy-on-Write).
