**`NSPrivateQueueConcurrencyType`** — это константа из **[[Core Data]]**, используемая при создании **[[NSManagedObjectContext]]**, которая определяет, что **контекст работает в собственной приватной (фоновой) очереди**.

- Контекст с этим типом **не блокирует главный поток**, что идеально для **тяжёлых операций**: массовая вставка, обновление или загрузка данных.
    
- Все действия с таким контекстом должны выполняться через **`perform {}`** или **`performAndWait {}`**, чтобы быть потокобезопасными.
    

Относится к: **Core Data → [[NSManagedObjectContext]] / Concurrency**

---

## 🔹 Примеры кода

### 1. Создание контекста для фоновых операций

```swift
import CoreData

let backgroundContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
```

- Контекст теперь **фоновый**, все изменения нужно выполнять в `perform` или `performAndWait`.
    

---

### 2. Присвоение родительского контекста

```swift
backgroundContext.parent = mainContext // mainContext — NSMainQueueConcurrencyType
```

- Изменения фонового контекста можно объединить в **главный контекст**, чтобы обновить UI.
    

---

### 3. Добавление объекта и сохранение в фоне

```swift
backgroundContext.perform {
    let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    user.setValue("Alice", forKey: "name")

    do {
        try backgroundContext.save()
        print("Объект User сохранён в фоне")
    } catch {
        print("Ошибка сохранения: \(error)")
    }
}
```

- Все операции выполняются **асинхронно** в приватной очереди.
    

---

### 4. Объединение изменений с главным контекстом

```swift
NotificationCenter.default.addObserver(forName: .NSManagedObjectContextDidSave, object: backgroundContext, queue: .main) { notification in
    mainContext.mergeChanges(fromContextDidSave: notification)
}
```

- Главный контекст ([[NSMainQueueConcurrencyType]]) получает изменения, и UI можно обновлять безопасно.
    

---

### 5. Использование с [[NSPersistentContainer]]

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

- `newBackgroundContext()` создаёт **контекст с приватной очередью** автоматически.
    
- Идеально для фоновых операций в приложении.
    

---

### 🔹 Отличие от [[NSMainQueueConcurrencyType]]

|Тип контекста|Очередь|Использование|
|---|---|---|
|`.mainQueueConcurrencyType`|Главная|UI операции, viewContext|
|`.privateQueueConcurrencyType`|Фоновая|Тяжёлые операции, batch insert/update|

- Для фонового контекста нельзя напрямую обновлять UI, нужно **синхронизировать с mainContext**.
    
