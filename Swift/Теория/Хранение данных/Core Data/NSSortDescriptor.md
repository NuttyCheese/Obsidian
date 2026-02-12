#save_data #swift #coredata
## 📘 Определение

**`NSSortDescriptor`** — класс из **[[Foundation]]**, используемый для **сортировки коллекций и выборок Core Data**.

- Определяет **ключ (ключевой путь) для сортировки**, направление (`ascending/descending`) и опционально **кастомное сравнение**.
    
- Часто применяется с **`NSArray`**, **`NSFetchRequest`** и **[[Core Data]]** для упорядочивания объектов.
    

Относится к: **Foundation / Core Data → Sorting**

---

## 🔹 Примеры кода

### 1. Сортировка массива строк

```swift
import Foundation

let names = ["Charlie", "Alice", "Bob"]
let sortDescriptor = NSSortDescriptor(key: nil, ascending: true) // key=nil для массива элементов
let sortedNames = (names as NSArray).sortedArray(using: [sortDescriptor])

print(sortedNames) // ["Alice", "Bob", "Charlie"]
```

---

### 2. Сортировка массива словарей по ключу

```swift
let users = [
    ["name": "Charlie", "age": 25],
    ["name": "Alice", "age": 30],
    ["name": "Bob", "age": 20]
]

let sortByName = NSSortDescriptor(key: "name", ascending: true)
let sortedUsers = (users as NSArray).sortedArray(using: [sortByName])

print(sortedUsers)
// [["name": "Alice", "age": 30], ["name": "Bob", "age": 20], ["name": "Charlie", "age": 25]]
```

---

### 3. Сортировка по числовому значению

```swift
let sortByAge = NSSortDescriptor(key: "age", ascending: false)
let sortedByAge = (users as NSArray).sortedArray(using: [sortByAge])

print(sortedByAge)
// [["name": "Alice", "age": 30], ["name": "Charlie", "age": 25], ["name": "Bob", "age": 20]]
```

---

### 4. Использование в Core Data fetch request

```swift
import CoreData

let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
let sortByName = NSSortDescriptor(key: "name", ascending: true)
let sortByAge = NSSortDescriptor(key: "age", ascending: false)

fetchRequest.sortDescriptors = [sortByName, sortByAge]

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

- Поддерживается **несколько сортировок**: сначала по `name`, затем по `age`.
    

---

### 5. Кастомная функция сравнения

```swift
let sortDescriptor = NSSortDescriptor(key: "name", ascending: true) { (obj1, obj2) -> ComparisonResult in
    guard let str1 = obj1 as? String, let str2 = obj2 as? String else { return .orderedSame }
    return str1.count < str2.count ? .orderedAscending : .orderedDescending
}

let sorted = (names as NSArray).sortedArray(using: [sortDescriptor])
print(sorted) // сортировка по длине имени
```

---

### 🔹 Особенности

1. Может использоваться с **NSArray**, **Core Data**, **NSFetchRequest**.
    
2. Поддерживает сортировку по **ключу, ключевому пути или кастомной функции**.
    
3. Можно комбинировать несколько `NSSortDescriptor` для сложных правил сортировки.
    
4. В Core Data сортировка выполняется на уровне запроса, что **эффективнее, чем сортировка в памяти**.
    
