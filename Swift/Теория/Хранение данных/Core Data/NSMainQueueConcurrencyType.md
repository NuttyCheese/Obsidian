#save_data #swift #coredata 
## 📘 Определение

**`NSMainQueueConcurrencyType`** — это константа из **[[Core Data]]**, используемая при создании **`NSManagedObjectContext`**, которая определяет **тип очереди контекста**.

Суть:

- Контекст с `NSMainQueueConcurrencyType` работает **в главной очереди (main thread)**.
    
- Предназначен для **обновления UI**, так как все изменения в UI должны выполняться на главном потоке.
    

Относится к: **Core Data → [[NSManagedObjectContext]] / Concurrency**

---

## 🔹 Примеры кода

### 1. Создание контекста для главного потока

```swift
import CoreData

let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
```

- `context` теперь работает **в главной очереди**.
    
- Можно безопасно использовать для обновления UI после fetch или save.
    

---

### 2. Использование с [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { _, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
}

let mainContext = container.viewContext // viewContext по умолчанию mainQueue
```

- `container.viewContext` **имеет NSMainQueueConcurrencyType**.
    
- Все fetch и save на этом контексте можно делать **на главном потоке**.
    

---

### 3. Fetch данных с использованием mainQueue

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
do {
    let users = try mainContext.fetch(fetchRequest)
    // Можно обновлять UI напрямую
    print("Найдено пользователей: \(users.count)")
} catch {
    print("Ошибка fetch: \(error)")
}
```

---

### 4. Сохранение изменений

```swift
let newUser = NSEntityDescription.insertNewObject(forEntityName: "User", into: mainContext)
newUser.setValue("Alice", forKey: "name")

do {
    try mainContext.save()
    print("Сохранили пользователя на главной очереди")
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 5. Отличие от `.privateQueueConcurrencyType`

|Тип контекста|Очередь|Использование|
|---|---|---|
|`.mainQueueConcurrencyType`|Главная|Для UI и работы с viewContext|
|`.privateQueueConcurrencyType`|Фоновая|Для фоновых операций (batch insert, heavy processing)|

---

**Совет:**

- Используйте **`NSMainQueueConcurrencyType`** для контекста, который работает с UI.
    
- Для **тяжелых или асинхронных операций** создавайте отдельный контекст с `.privateQueueConcurrencyType` и объединяйте изменения через `mergeChanges(fromContextDidSave:)`.
    
