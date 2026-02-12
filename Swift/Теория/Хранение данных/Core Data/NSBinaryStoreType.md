#save_data #Swift 
## 📘 Определение

**`NSBinaryStoreType`** — это **тип физического хранилища в [[Core Data]]**, который сохраняет данные **в бинарном формате на диск**.

- В отличие от **[[SQLite]]**, данные хранятся **как один бинарный файл**, полностью управляемый Core Data.
    
- Подходит для **малых и средних объёмов данных**, когда нужна компактность и простота.
    
- Поддерживает все возможности Core Data: **атрибуты, связи, транзакции**, но **не масштабируется на большие объёмы**, как SQLite.
    

Относится к: **Core Data → [[Persistence]] / Store**

---

## 🔹 Примеры кода

### 1. Создание Binary хранилища вручную

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let model = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: model)

let storeURL = FileManager.default.temporaryDirectory.appendingPathComponent("store.data")

do {
    try coordinator.addPersistentStore(ofType: NSBinaryStoreType,
                                       configurationName: nil,
                                       at: storeURL)
    print("Binary хранилище создано: \(storeURL)")
} catch {
    fatalError("Не удалось создать Binary хранилище: \(error)")
}
```

---

### 2. Использование с [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
let description = NSPersistentStoreDescription()
description.type = NSBinaryStoreType
description.url = FileManager.default.temporaryDirectory.appendingPathComponent("store.data")

container.persistentStoreDescriptions = [description]
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки Binary store: \(error)")
    }
    print("Binary store загружен: \(storeDescription.url?.absoluteString ?? "")")
}
```

---

### 3. Сохранение объекта в Binary Store

```swift
let context = container.viewContext
let user = User(context: context)
user.name = "Alice"
user.age = 25

do {
    try context.save()
    print("Объект сохранён в Binary store")
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 4. Чтение данных из Binary Store

```swift
let fetchRequest = NSFetchRequest<User>(entityName: "User")

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.name) }
} catch {
    print("Ошибка fetch: \(error)")
}
```

---

### 5. Удаление Binary Store

```swift
if let store = container.persistentStoreCoordinator.persistentStores.first {
    try container.persistentStoreCoordinator.remove(store)
    print("Binary store удалён")
}
```

- После удаления можно создать новое Binary хранилище или очистить данные.
    

---

### 🔹 Особенности

|Свойство / Тип|Описание|
|---|---|
|`NSBinaryStoreType`|Тип хранилища Core Data — бинарный файл|
|`storeURL`|Путь к бинарному файлу|
|Применение|Малые/средние объёмы данных, тесты, прототипы|
|Скорость|Быстрое чтение/запись|
|Ограничения|Не подходит для больших баз данных, как SQLite|

- **NSBinaryStoreType** — удобен для компактного хранения Core Data объектов.
    
- Для больших данных лучше использовать **[[NSSQLiteStoreType]]**.
    
