#save_data #swift #coredata 
## 📘 Определение

**`privateQueueConcurrencyType`** в [[Core Data]] — это тип контекста ([[NSManagedObjectContext]]), который **работает в собственной приватной (фоновой) очереди**.

- Используется для **тяжёлых операций с данными**, которые не должны блокировать главный поток (`main thread`).
    
- Все действия с таким контекстом должны выполняться через **`perform {}`** или **`performAndWait {}`**, чтобы быть потокобезопасными.
    
- Позволяет использовать **фоновые контексты** для массовой вставки, обновления или удаления объектов, а затем объединять изменения с главным контекстом.
    

Относится к: **Core Data → [[NSManagedObjectContext]] / Concurrency**

---

## 🔹 Примеры кода

### 1. Создание приватного контекста

```swift
import CoreData

let backgroundContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
backgroundContext.parent = mainContext // mainContext — NSMainQueueConcurrencyType
```

- Контекст будет работать в **фоновой очереди**.
    
- Родительский контекст нужен для слияния изменений с главным потоком.
    

---

### 2. Выполнение операций с объектами в фоне

```swift
backgroundContext.perform {
    let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    user.setValue("Alice", forKey: "name")
    
    do {
        try backgroundContext.save() // сохраняем изменения в приватный контекст
        print("Объект сохранён в фоне")
    } catch {
        print("Ошибка сохранения: \(error)")
    }
}
```

- Все действия выполняются **асинхронно**, не блокируя UI.
    

---

### 3. Слияние изменений с главным контекстом

```swift
NotificationCenter.default.addObserver(forName: .NSManagedObjectContextDidSave, object: backgroundContext, queue: .main) { notification in
    mainContext.mergeChanges(fromContextDidSave: notification)
}
```

- Главный контекст ([[NSMainQueueConcurrencyType]]) получает изменения, и UI можно обновлять безопасно.
    

---

### 4. Создание фонового контекста через [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { _, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
}

let bgContext = container.newBackgroundContext() // автоматически privateQueue
bgContext.perform {
    let newUser = User(context: bgContext)
    newUser.name = "Bob"
    try? bgContext.save()
}
```

- `newBackgroundContext()` автоматически создаёт **privateQueue** контекст для фоновых операций.
    

---

### 5. Использование `performAndWait`

```swift
backgroundContext.performAndWait {
    let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    user.setValue("Charlie", forKey: "name")
    try? backgroundContext.save()
}
```

- `performAndWait` выполняет блок синхронно в приватной очереди, блокируя текущий поток до завершения.
    

---

### 🔹 Отличие от mainQueue

|Тип контекста|Очередь|Назначение|
|---|---|---|
|`.mainQueueConcurrencyType`|Главная (UI)|Для UI операций|
|`.privateQueueConcurrencyType`|Фоновая|Тяжёлые операции с данными, не блокирующие UI|

- Контексты **privateQueue** нельзя использовать напрямую для обновления UI.
    
- Нужно **синхронизировать с mainContext**, чтобы изменения были видны на интерфейсе.
    
