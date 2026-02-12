#save_data #swift #coredata 
## 📘 Определение

**`NSManagedObjectContext`** — основной класс **Core Data**, который управляет **объектами `NSManagedObject`**.

- Контекст — это **“рабочая область”**, где создаются, изменяются, удаляются и читаются объекты [[Core Data]].
    
- Он отвечает за **сохранение изменений в хранилище ([[NSPersistentStore]])** и управление **состоянием объектов**.
    
- Может быть **главным (`mainQueue`)** или **фоновым (`privateQueue`)**, в зависимости от типа очереди.
    

Относится к: **Core Data → [[Context]] / [[Persistence]]**

---

## 🔹 Примеры кода

### 1. Создание контекста вручную

```swift
import CoreData

let mainContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
```

- Контекст с `.mainQueueConcurrencyType` безопасен для **UI операций**.
    

---

### 2. Создание контекста через [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { _, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
}
let context = container.viewContext // mainQueue
```

---

### 3. Создание и сохранение объекта

```swift
let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")
user.setValue(25, forKey: "age")

do {
    try context.save()
    print("Объект User сохранён")
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 4. Fetch объектов из контекста

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
do {
    let users = try context.fetch(fetchRequest)
    for user in users {
        print(user.value(forKey: "name") ?? "Нет имени")
    }
} catch {
    print("Ошибка fetch: \(error)")
}
```

---

### 5. Создание фонового контекста ([[privateQueue]])

```swift
let backgroundContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
backgroundContext.parent = context // родитель — mainContext

backgroundContext.perform {
    let newUser = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    newUser.setValue("Bob", forKey: "name")

    do {
        try backgroundContext.save()
        try context.save() // сохранение в mainContext
    } catch {
        print("Ошибка сохранения в фоне: \(error)")
    }
}
```

- Используется для **тяжёлых операций**, чтобы не блокировать UI.
    
- Изменения объединяются через **родительский контекст**.
    

---

### 🔹 Основные особенности

|Свойство / Метод|Назначение|
|---|---|
|`perform(_:)` / `performAndWait(_:)`|Безопасное выполнение операций в контексте|
|`save()`|Сохраняет изменения в хранилище или родительский контекст|
|`parent`|Родительский контекст для иерархической синхронизации|
|`fetch(_:)`|Получение объектов из хранилища|
|`delete(_:)`|Удаление объекта из контекста|
