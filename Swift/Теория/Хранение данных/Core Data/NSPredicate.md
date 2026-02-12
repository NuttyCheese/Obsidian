#save_data #swift #coredata 
## 📘 Определение

**`NSPredicate`** — класс из **[[Foundation]]**, используемый для **фильтрации данных** и создания условий выборки.

- Часто применяется с **Core Data** (`NSFetchRequest`), массивами (`filter`) и `NSArray`.
    
- Позволяет описывать **логические условия** для поиска объектов по атрибутам.
    
- Использует **синтаксис предикатов**, похожий на SQL.
    

Относится к: **[[Foundation]] → Filtering / [[Core Data]]**

---

## 🔹 Примеры кода

### 1. Фильтрация массива строк

```swift
import Foundation

let names = ["Alice", "Bob", "Charlie", "David"]
let predicate = NSPredicate(format: "SELF BEGINSWITH[c] %@", "A")
let filtered = names.filter { predicate.evaluate(with: $0) }

print(filtered) // ["Alice"]
```

- `SELF` — текущий элемент массива.
    
- `[c]` — игнорировать регистр.
    

---

### 2. Простое сравнение с числом

```swift
let numbers = [1, 5, 10, 15]
let predicate = NSPredicate(format: "SELF > %d", 5)
let filtered = numbers.filter { predicate.evaluate(with: $0) }

print(filtered) // [10, 15]
```

---

### 3. Использование с Core Data

```swift
import CoreData

let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
fetchRequest.predicate = NSPredicate(format: "age >= %d AND name CONTAINS[c] %@", 18, "a")

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

- Фильтруем пользователей старше 18 лет с буквой "a" в имени.
    
- Используется **безопасное подставление параметров** (`%@`, `%d`).
    

---

### 4. Фильтрация массива словарей

```swift
let users = [
    ["name": "Alice", "age": 25],
    ["name": "Bob", "age": 17],
    ["name": "Charlie", "age": 30]
]

let predicate = NSPredicate(format: "age >= %d", 18)
let filtered = (users as NSArray).filtered(using: predicate)

print(filtered) 
// [["name": "Alice", "age": 25], ["name": "Charlie", "age": 30]]
```

---

### 5. Сложные условия с OR и IN

```swift
let predicate = NSPredicate(format: "name IN %@ OR age < %d", ["Alice", "Bob"], 20)
let filtered = (users as NSArray).filtered(using: predicate)

print(filtered)
// [["name": "Alice", "age": 25], ["name": "Bob", "age": 17], ["name": "Charlie", "age": 30]]
```

- `IN` проверяет наличие значения в массиве.
    
- `OR`, `AND`, `NOT` — поддерживаются для логических выражений.
    

---

### 🔹 Особенности

1. Используется для **массивов, Core Data и других коллекций**.
    
2. Поддерживает **форматные спецификаторы** (`%@`, `%d`, `%f`).
    
3. Позволяет **безопасно подставлять значения**, избегая SQL-инъекций в Core Data.
    
4. Можно комбинировать с **sortDescriptors** для сортировки результатов.
    
