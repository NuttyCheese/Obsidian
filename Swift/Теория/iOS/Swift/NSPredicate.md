**`NSPredicate`** — это мощный и гибкий инструмент в Foundation для создания **логических условий фильтрации** объектов.  
Он используется в основном с:

- **Core Data** (`NSFetchRequest.predicate`)
- **NSArray** / `NSArray.filtered(using:)`
- **Swift коллекциями** через `filter { predicate.evaluate(with: $0) }`

NSPredicate позволяет описывать сложные запросы в **строковом формате**, похожем на SQL, но с полной типобезопасностью и безопасной подстановкой параметров.

### 1. Почему NSPredicate всё ещё актуален в 2025–2026

| Сценарий                                      | Почему NSPredicate здесь лучший выбор                  | Альтернатива (когда можно обойтись без него) |
|-----------------------------------------------|--------------------------------------------------------|----------------------------------------------|
| Core Data fetch request                       | Самый надёжный и безопасный способ фильтрации          | SwiftData / @Query (если используешь SwiftData) |
| Фильтрация NSArray / NSMutableArray           | Работает нативно, поддерживает KVC                     | `filter` с замыканием                        |
| Сложные условия (AND/OR/IN/BEGINSWITH и т.д.) | Очень выразительный синтаксис                          | `filter` + цепочка условий                   |
| Динамическая фильтрация (поиск по тексту)     | Легко строить предикаты в runtime                      | Combine / @Query                             |
| Legacy-код / Objective-C interop              | NSPredicate — стандарт Foundation                      | —                                            |

### 2. Самые важные форматы NSPredicate (шпаргалка 2026)

| Формат / Оператор                  | Описание                                               | Пример NSPredicate(format:)                     | Эквивалент в Swift замыкании |
|------------------------------------|--------------------------------------------------------|-------------------------------------------------|--------------------------------|
| `==`, `=`, `!=`                    | Равенство / неравенство                                | `"age == %d"`                                   | `$0.age == 25`                 |
| `>`, `<`, `>=`, `<=`               | Сравнение чисел                                        | `"price > 100"`                                 | `$0.price > 100`               |
| `IN`                               | Вхождение в массив/множество                           | `"status IN %@"`                                | `["active", "pending"].contains($0.status)` |
| `BETWEEN`                          | Диапазон                                               | `"age BETWEEN {18, 65}"`                        | `$0.age >= 18 && $0.age <= 65` |
| `BEGINSWITH[c]`, `ENDSWITH[c]`     | Начинается/заканчивается (игнорируя регистр)           | `"name BEGINSWITH[c] %@"`                       | `$0.name.lowercased().hasPrefix("a")` |
| `CONTAINS[c]`, `LIKE[c]`           | Содержит / wildcard                                    | `"name CONTAINS[c] %@"`                         | `$0.name.lowercased().contains("bob")` |
| `ANY`, `ALL`                       | Для коллекций (to-many отношения)                      | `"ANY tags.name == %@"`                         | —                              |
| `&&`, `AND`, `||`, `OR`, `NOT`     | Логические операторы                                   | `"age > 18 AND status == 'active'"`             | `$0.age > 18 && $0.status == "active"` |

### 3. Самый безопасный и современный паттерн 2026 (Core Data + NSPredicate)

```swift
final class UserRepository {
    private let context: NSManagedObjectContext
    
    init(context: NSManagedObjectContext) {
        self.context = context
    }
    
    func fetchActiveUsers(olderThan minAge: Int, searchText: String?) throws -> [User] {
        let request = User.fetchRequest()
        
        var subpredicates: [NSPredicate] = [
            NSPredicate(format: "isActive == true"),
            NSPredicate(format: "age >= %d", minAge)
        ]
        
        if let searchText, !searchText.isEmpty {
            let searchPredicate = NSPredicate(format: "name CONTAINS[c] %@ OR email CONTAINS[c] %@", searchText, searchText)
            subpredicates.append(searchPredicate)
        }
        
        request.predicate = NSCompoundPredicate(andPredicateWithSubpredicates: subpredicates)
        
        // Сортировка
        request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        return try context.fetch(request)
    }
}
```

### 4. Лучшие практики NSPredicate в 2026

- **Всегда** используй **форматированные строки с подстановкой** (`%@`, `%d`, `%f`) — это **защищает** от инъекций  
- **Никогда** не конкатенируй строки вручную: `"name == '\(search)'"` → уязвимость и ошибки с кавычками  
- **Для нескольких условий** — используй `NSCompoundPredicate` (`andPredicateWithSubpredicates`, `orPredicateWithSubpredicates`)  
- **Для поиска по тексту** — всегда добавляй `[c]` (case-insensitive) и `[d]` (diacritic-insensitive)  
  ```swift
  NSPredicate(format: "name CONTAINS[cd] %@", searchText)
  ```
- **В SwiftUI / Combine** — предпочитай `@Query` или `filter` замыканиями вместо NSPredicate  
- **Для производительности** — используй индексы в модели Core Data для часто фильтруемых атрибутов  
- **Документируйте** — пиши комментарий «NSPredicate — поиск активных пользователей старше minAge с текстом в имени или email»

**Короткий девиз 2026**:
> `NSPredicate` — это «SQL-подобный язык запросов» внутри Swift для Core Data и NSArray.  
> В 2026 году:  
> - используй **форматированные строки** с `%@`, `%d`  
> - `[c]` / `[cd]` — для поиска без учёта регистра и диакритик  
> - `NSCompoundPredicate` — для сложных AND/OR условий  
> - в новом коде → чаще `filter` замыканиями или `@Query`  
> Это **основа** мощной и безопасной фильтрации в Core Data.

Удачи с быстрыми и точными запросами к данным! 🔍