#save_data #swift #coredata 
## 📘 Определение

**`context` в [[Core Data]]** — это сокращённое и привычное название **[[NSManagedObjectContext]]**, основного объекта, который **управляет жизненным циклом объектов Core Data**.

- Контекст — это **“рабочая область”** для создания, изменения, удаления и извлечения [[NSManagedObject]].
    
- Все операции с объектами Core Data выполняются через контекст.
    
- Контекст **отвечает за транзакции**, сохранение изменений в хранилище ([[NSPersistentStore]]) и синхронизацию данных между потоками.
    

Относится к: **Core Data → NSManagedObjectContext / [[Persistence]]**

---

## 🔹 Примеры кода

### 1. Получение контекста из `NSPersistentContainer`

```swift
import CoreData

let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { _, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
}

let context = container.viewContext // mainQueue context
```

- `viewContext` — контекст для главного потока, безопасен для UI.
    

---

### 2. Создание объекта через контекст

```swift
let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")
user.setValue(25, forKey: "age")

do {
    try context.save() // сохраняем изменения в хранилище
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 3. Fetch объектов через контекст

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

---

### 4. Фоновый контекст и perform

```swift
let bgContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
bgContext.parent = context

bgContext.perform {
    let newUser = NSEntityDescription.insertNewObject(forEntityName: "User", into: bgContext)
    newUser.setValue("Bob", forKey: "name")

    try? bgContext.save()           // сохраняем в bgContext
    try? context.save()             // объединяем изменения в главный context
}
```

- `perform {}` гарантирует, что все действия **безопасны для фоновой очереди**.
    

---

### 5. Отслеживание изменений через `NSManagedObjectContextDidSaveNotification`

```swift
NotificationCenter.default.addObserver(forName: .NSManagedObjectContextDidSave, object: nil, queue: .main) { notification in
    context.mergeChanges(fromContextDidSave: notification)
}
```

- Используется для **синхронизации фоновых контекстов с главным контекстом**, чтобы UI был актуален.
    

---

### 🔹 Особенности

|Свойство / Метод|Назначение|
|---|---|
|`save()`|Сохраняет изменения в хранилище или родительский контекст|
|`fetch(_:)`|Получение объектов из Core Data|
|`delete(_:)`|Удаление объекта из контекста|
|`perform {}` / `performAndWait {}`|Безопасное выполнение операций на контексте|
|`parent`|Родительский контекст для синхронизации фоновых операций|

- Контекст управляет **жизненным циклом объектов**, транзакциями и **синхронизацией между потоками**.
    
- Использование нескольких контекстов (главный + фоновые) улучшает производительность и предотвращает блокировку UI.
    
