**`NSSortDescriptor`** — это класс из Foundation, который определяет **одно правило сортировки** для коллекций объектов или результатов выборки в Core Data.

Он позволяет указать:
- по какому **ключу** (атрибуту) сортировать,
- в каком **направлении** (возрастание/убывание),
- и опционально — **кастомную функцию сравнения**.

Самый частый сценарий использования — сортировка в `NSFetchRequest` (Core Data) и сортировка массивов/массивов словарей через `sortedArray(using:)`.

### 1. Основные свойства и конструкторы

```swift
// Самый частый конструктор
NSSortDescriptor(key: String?, ascending: Bool)

// С кастомной функцией сравнения
NSSortDescriptor(key: String?, ascending: Bool, selector: Selector)

// С замыканием (блоком сравнения)
NSSortDescriptor(key: String?, ascending: Bool, comparator: @escaping (Any, Any) -> ComparisonResult)
```

### 2. Самые популярные паттерны 2026

#### 2.1 Сортировка в Core Data (самый частый случай)

```swift
let request = NSFetchRequest<User>(entityName: "User")

// Сортировка по имени (A→Z), затем по возрасту (по убыванию)
let sortByName = NSSortDescriptor(key: "name", ascending: true)
let sortByAgeDesc = NSSortDescriptor(key: "age", ascending: false)

request.sortDescriptors = [sortByName, sortByAgeDesc]

let users = try context.fetch(request)
// Результат: сначала по алфавиту имени, при равных именах — старшие сверху
```

#### 2.2 Сортировка массива структур по нескольким полям

```swift
struct Product {
    let name: String
    let price: Double
    let category: String
}

let products: [Product] = [...] 

let sortByCategory = NSSortDescriptor(key: "category", ascending: true)
let sortByPriceDesc = NSSortDescriptor(key: "price", ascending: false)

let sorted = (products as NSArray).sortedArray(using: [sortByCategory, sortByPriceDesc]) as! [Product]

// Сначала по категории (A→Z), внутри категории — по цене от дорогих к дешёвым
```

#### 2.3 Сортировка по длине строки (кастомный компаратор)

```swift
let names = ["Alexander", "Bob", "Charlie", "David"]

let sortByLengthDesc = NSSortDescriptor(key: nil, ascending: false) { obj1, obj2 in
    guard let str1 = obj1 as? String, let str2 = obj2 as? String else {
        return .orderedSame
    }
    return str1.count > str2.count ? .orderedAscending : .orderedDescending
}

let sorted = (names as NSArray).sortedArray(using: [sortByLengthDesc]) as! [String]
// → ["Alexander", "Charlie", "David", "Bob"]
```

#### 2.4 Сортировка с учётом локали (case-insensitive, diacritic-insensitive)

```swift
let sortDescriptor = NSSortDescriptor(key: "name", ascending: true) {
    (obj1, obj2) -> ComparisonResult in
    
    guard let str1 = obj1 as? String, let str2 = obj2 as? String else {
        return .orderedSame
    }
    
    return str1.localizedStandardCompare(str2)
}

let names = ["Émilie", "alice", "Bob", "çédric"]
let sorted = (names as NSArray).sortedArray(using: [sortDescriptor]) as! [String]
// → ["alice", "Bob", "çédric", "Émilie"] (правильный порядок с учётом локали)
```

### 3. Лучшие практики NSSortDescriptor в 2026

- **Всегда** используй **массив** `sortDescriptors` — даже если сортировка по одному полю  
- **Для Core Data** — сортируй **на уровне запроса** (`fetchRequest.sortDescriptors`), а не в памяти  
- **Для case-insensitive поиска** — используй `localizedStandardCompare` или `[c]` в предикатах  
- **Для сложной сортировки** — предпочитай кастомный компаратор или `NSSortDescriptor` с замыканием  
- **Не используй** `NSSortDescriptor` для больших массивов в памяти — лучше `sorted(by:)` с замыканием  
- **Swift 6 strict concurrency** — `NSSortDescriptor` полностью безопасен, но сам fetch должен быть на правильном контексте  
- **Документируйте** — пиши комментарий «Сортировка: сначала по категории (A→Z), затем по цене (desc)»

**Короткий девиз 2026**:
> `NSSortDescriptor` — это «правило сортировки» для Core Data и NSArray.  
> В 2026 году:  
> - используй в `NSFetchRequest.sortDescriptors` для Core Data  
> - для массивов — чаще `sorted(by:)` с замыканием  
> - кастомный компаратор — только для сложной логики (локаль, длина и т.д.)  
> - массив дескрипторов — стандарт для нескольких полей сортировки  

Удачи с идеально отсортированными данными в твоём приложении! 📊