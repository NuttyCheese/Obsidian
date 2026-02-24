**`NSSQLiteStoreType`** — это **константа из [[Core Data]]**, указывающая тип **физического хранилища** для [[NSPersistentStore]].

- Используется при добавлении хранилища в **[[NSPersistentStoreCoordinator]]**.
    
- Фактически представляет **[[SQLite]]-базу**, где Core Data хранит все объекты и их связи.
    
- Подходит для большинства приложений, т.к. обеспечивает **постоянное хранение на диске** и поддерживает большие объёмы данных.
    

Относится к: **Core Data → [[Persistence]] / Store**

---

## 🔹 Примеры кода

### 1. Добавление SQLite хранилища к координатору

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let managedObjectModel = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)

let storeURL = URL(fileURLWithPath: "/path/to/store.sqlite")

do {
    let store = try coordinator.addPersistentStore(ofType: NSSQLiteStoreType,
                                                   configurationName: nil,
                                                   at: storeURL)
    print("SQLite хранилище добавлено: \(store)")
} catch {
    fatalError("Не удалось добавить хранилище: \(error)")
}
```

---

### 2. Использование с [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки SQLite хранилища: \(error)")
    }
    print("SQLite хранилище загружено: \(storeDescription.url?.absoluteString ?? "")")
}
```

- По умолчанию `NSPersistentContainer` использует `NSSQLiteStoreType` для постоянного хранения.
    

---

### 3. Настройка опций для SQLite хранилища

```swift
let options = [
    NSMigratePersistentStoresAutomaticallyOption: true,
    NSInferMappingModelAutomaticallyOption: true
]

do {
    try coordinator.addPersistentStore(ofType: NSSQLiteStoreType,
                                       configurationName: nil,
                                       at: storeURL,
                                       options: options)
    print("SQLite хранилище с опциями добавлено")
} catch {
    fatalError("Ошибка добавления хранилища: \(error)")
}
```

- `NSMigratePersistentStoresAutomaticallyOption` — автоматическая миграция.
    
- `NSInferMappingModelAutomaticallyOption` — автоматическое определение схемы для миграции.
    

---

### 4. Проверка существующего SQLite хранилища

```swift
if FileManager.default.fileExists(atPath: storeURL.path) {
    print("Файл SQLite хранилища существует")
} else {
    print("Хранилище ещё не создано")
}
```

---

### 🔹 Особенности

|Свойство / Опция|Описание|
|---|---|
|`NSSQLiteStoreType`|Тип хранилища SQLite|
|`storeURL`|Путь к файлу на диске|
|`options`|Настройки миграции, журналирования и защиты данных|
|Поддержка больших данных|SQLite масштабируется и подходит для реальных приложений|
|Интеграция с NSPersistentContainer|Используется по умолчанию|

- **NSSQLiteStoreType** — оптимальный выбор для **постоянного хранения данных** Core Data.
    
- Для тестов и временных данных можно использовать **[[NSInMemoryStoreType]]**.
    
