#save_data #Swift
## 📘 Определение

**`NSInMemoryStoreType`** — это тип хранилища в [[Core Data]], который сохраняет данные **только в оперативной памяти (RAM)**, без записи на диск.

- Данные **теряются после закрытия приложения**.
    
- Используется для **тестов, временного кеша** или когда не нужен постоянный storage.
    
- Позволяет работать с Core Data объектами через обычный контекст ([[NSManagedObjectContext]]) с максимальной скоростью.
    

Относится к: **Core Data → [[Persistence]] / Store**

---

## 🔹 Примеры кода

### 1. Создание [[In-Memory]] хранилища вручную

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let model = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: model)

do {
    try coordinator.addPersistentStore(ofType: NSInMemoryStoreType,
                                       configurationName: nil,
                                       at: nil)
    print("In-Memory хранилище создано")
} catch {
    fatalError("Не удалось создать In-Memory store: \(error)")
}
```

- `at: nil` — указывает, что хранилище **не будет создано на диске**.
    

---

### 2. Использование с [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
let description = NSPersistentStoreDescription()
description.type = NSInMemoryStoreType

container.persistentStoreDescriptions = [description]
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки In-Memory store: \(error)")
    }
    print("In-Memory store загружен")
}
```

- Идеально для **юнит-тестов**, когда не нужно сохранять реальные данные.
    

---

### 3. Сохранение объекта в In-Memory store

```swift
let context = container.viewContext
let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")

try? context.save() // изменения остаются только в памяти
```

---

### 4. Получение данных из In-Memory store

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

- Работает как обычный fetch, но данные находятся только в RAM.
    

---

### 5. Использование In-Memory для тестов

```swift
func testInsertUser() {
    let container = NSPersistentContainer(name: "Model")
    let description = NSPersistentStoreDescription()
    description.type = NSInMemoryStoreType
    container.persistentStoreDescriptions = [description]
    container.loadPersistentStores { _, _ in }

    let context = container.viewContext
    let user = User(context: context)
    user.name = "Bob"
    try? context.save()

    let fetchRequest = NSFetchRequest<User>(entityName: "User")
    let users = try? context.fetch(fetchRequest)
    assert(users?.count == 1)
}
```

- После завершения теста данные **не сохраняются на диске**.
    

---

### 🔹 Особенности

|Свойство / Тип|Описание|
|---|---|
|`NSInMemoryStoreType`|Хранилище Core Data в памяти|
|Сохранность|Данные теряются после закрытия приложения|
|Применение|Unit-тесты, временный кеш, прототипы|
|Скорость|Максимальная скорость операций|
|Ограничения|Не подходит для постоянного хранения больших объёмов данных|

- **NSInMemoryStoreType** — оптимальное решение для **временных данных и тестирования**, не требующих постоянного сохранения.
    
