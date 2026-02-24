**`NSPersistentContainer`** — это класс из **[[Core Data]]**, который **упрощает настройку и управление стеком Core Data**.

- Объединяет **[[NSManagedObjectModel]]**, **[[NSPersistentStoreCoordinator]]** и **[[NSManagedObjectContext]]** в один объект.
    
- Облегчает **создание контекста, загрузку хранилища и работу с объектами**.
    
- Рекомендуется использовать с iOS 10 и выше для стандартного управления [[Core Data]].
    

Относится к: **Core Data → Stack / [[Persistence]]**

---

## 🔹 Примеры кода

### 1. Создание контейнера

```swift
import CoreData

let container = NSPersistentContainer(name: "Model")
```

- `"Model"` — имя `.xcdatamodeld` файла.
    
- Контейнер пока не загружает хранилище.
    

---

### 2. Загрузка хранилища

```swift
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
    print("Хранилище загружено: \(storeDescription.url?.absoluteString ?? "")")
}
```

- Метод асинхронный, вызывает completion после загрузки.
    
- Контейнер готов к работе с контекстами.
    

---

### 3. Доступ к main context

```swift
let context = container.viewContext
```

- `viewContext` имеет **[[NSMainQueueConcurrencyType]]**, безопасен для работы с UI.
    

---

### 4. Создание фонового контекста

```swift
let backgroundContext = container.newBackgroundContext()

backgroundContext.perform {
    let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: backgroundContext)
    user.setValue("Alice", forKey: "name")
    
    do {
        try backgroundContext.save()
    } catch {
        print("Ошибка сохранения в фоне: \(error)")
    }
}
```

- Позволяет делать **тяжелые операции** без блокировки UI.
    
- Изменения можно объединить с `viewContext`.
    

---

### 5. Сохранение main context

```swift
func saveContext() {
    let context = container.viewContext
    if context.hasChanges {
        do {
            try context.save()
            print("Данные сохранены")
        } catch {
            print("Ошибка сохранения: \(error)")
        }
    }
}
```

- Всегда проверяйте `hasChanges` перед сохранением, чтобы избежать лишних операций.
    

---

### 🔹 Основные преимущества `NSPersistentContainer`

| Возможность                       | Описание                                                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `viewContext`                     | Контекст для main thread, UI операции                                                                                                |
| `newBackgroundContext()`          | Фоновый контекст для тяжелых операций                                                                                                |
| Автоматическая загрузка хранилища | Нет необходимости вручную создавать [[NSPersistentStoreCoordinator]]                                                                 |
| Легкая интеграция                 | Подходит для [[MVVM (Model-View-ViewModel) Architecture\|MVVM]], [[Clean Swift (VIP) Architecture\|Clean Swift]] и других архитектур |
| Управление версиями               | Совместим с миграциями Core Data                                                                                                     |
