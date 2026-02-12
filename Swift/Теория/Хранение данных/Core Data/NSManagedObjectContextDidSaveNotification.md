#save_data #swift #coredata 
## 📘 Определение

**`NSManagedObjectContextDidSaveNotification`** — это уведомление (**Notification**) из **[[Core Data]]**, которое отправляется, когда **[[NSManagedObjectContext]] успешно сохраняет изменения** в свой контекст или родительский контекст.

- Позволяет другим контекстам или частям приложения **узнать об изменениях** и при необходимости **синхронизировать данные**.
    
- Часто используется при работе с **фоновыми контекстами** и **объединении изменений** в главный контекст.
    

Относится к: **Core Data → Notifications / Context Synchronization**

---

## 🔹 Примеры кода

### 1. Подписка на уведомление

```swift
import CoreData

NotificationCenter.default.addObserver(
    self,
    selector: #selector(contextDidSave(_:)),
    name: NSNotification.Name.NSManagedObjectContextDidSave,
    object: nil
)
```

---

### 2. Обработчик уведомления

```swift
@objc func contextDidSave(_ notification: Notification) {
    guard let context = notification.object as? NSManagedObjectContext else { return }
    print("Контекст \(context) успешно сохранился")
    
    // Можно объединить изменения в другой контекст
    mainContext.perform {
        self.mainContext.mergeChanges(fromContextDidSave: notification)
    }
}
```

- `mergeChanges(fromContextDidSave:)` объединяет **все вставленные, изменённые и удалённые объекты** в целевой контекст.
    

---

### 3. Использование с фоновым контекстом

```swift
let backgroundContext = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
backgroundContext.parent = mainContext

backgroundContext.perform {
    let newUser = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    newUser.setValue("Alice", forKey: "name")

    do {
        try backgroundContext.save()
        // NSManagedObjectContextDidSaveNotification будет отправлено автоматически
    } catch {
        print("Ошибка сохранения: \(error)")
    }
}
```

- Главный контекст может слушать уведомление и **объединять изменения**, чтобы UI был актуален.
    

---

### 4. Применение в [[iOS]] UI

```swift
NotificationCenter.default.addObserver(forName: .NSManagedObjectContextDidSave, object: nil, queue: .main) { notification in
    mainContext.mergeChanges(fromContextDidSave: notification)
    tableView.reloadData() // обновляем UI
}
```

- Сохраняем данные в фоновом контексте.
    
- Главный контекст автоматически получает изменения и UI обновляется.
    

---

### 🔹 Важные моменты

1. **Отправляется только после успешного `save()`** контекста.
    
2. Содержит словарь с ключами: `inserted`, `updated`, `deleted` для объектов.
    
3. Используется при работе с **многоуровневыми контекстами** (main → background).
    
4. Позволяет **синхронизировать UI и данные без конфликтов**.
    
