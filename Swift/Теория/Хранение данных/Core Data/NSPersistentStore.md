**`NSPersistentStore`** — это класс из **[[Core Data]]**, представляющий **физическое хранилище данных** (например, [[SQLite]], [[Binary]] или [[In-Memory]]).

- Он хранит все объекты Core Data ([[NSManagedObject]]) **на диске или в памяти**.
    
- Управляется **[[NSPersistentStoreCoordinator]]**, который связывает **модель данных ([[NSManagedObjectModel]])** с конкретным хранилищем.
    
- Core Data поддерживает несколько типов хранилищ одновременно, что позволяет, например, иметь основной SQLite и временный in-memory store.
    

Относится к: **Core Data → [[Persistence]] / Store**

---

## 🔹 Примеры кода

### 1. Создание SQLite хранилища вручную

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let managedObjectModel = NSManagedObjectModel(contentsOf: modelURL)!

let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
let storeURL = URL(fileURLWithPath: "/path/to/store.sqlite")

do {
    let store = try coordinator.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: storeURL)
    print("Хранилище добавлено: \(store)")
} catch {
    fatalError("Не удалось добавить хранилище: \(error)")
}
```

- [[NSSQLiteStoreType]] — тип SQLite.
    
- Можно использовать другие типы: [[NSInMemoryStoreType]], [[NSBinaryStoreType]].
    

---

### 2. Создание in-memory хранилища

```swift
do {
    let store = try coordinator.addPersistentStore(ofType: NSInMemoryStoreType, configurationName: nil, at: nil)
    print("In-memory хранилище создано: \(store)")
} catch {
    fatalError("Ошибка создания in-memory хранилища: \(error)")
}
```

- Полезно для **тестов**, когда не нужно сохранять данные на диск.
    

---

### 3. Использование с [[NSPersistentContainer]]

```swift
import CoreData

let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки хранилища: \(error)")
    }
    print("Хранилище загружено: \(storeDescription.type)")
}
```

- `storeDescription.type` показывает тип хранилища (`SQLite`, `InMemory` и т.д.).
    

---

### 4. Проверка всех добавленных хранилищ

```swift
for store in coordinator.persistentStores {
    print("Store: \(store.url?.absoluteString ?? "In-memory")")
}
```

- Можно получить список всех подключённых хранилищ.
    

---

### 5. Удаление хранилища

```swift
if let store = coordinator.persistentStores.first {
    try coordinator.remove(store)
    print("Хранилище удалено")
}
```

- Используется при **тестах** или при **сбросе базы данных**.
    

---

### 🔹 Особенности

| Свойство / Метод    | Назначение                                                |
| ------------------- | --------------------------------------------------------- |
| `type`              | Тип хранилища: SQLite, [[Binary]], [[In-Memory]]          |
| `url`               | Путь к файлу на диске (для SQLite/Binary)                 |
| `configurationName` | Конфигурация модели для выборочного хранения              |
| `options`           | Параметры хранилища (миграции, автоматическое обновление) |

- **NSPersistentStore** — это физический уровень Core Data.
    
- **Координатор** ([[NSPersistentStoreCoordinator]]) управляет всеми хранилищами и связывает их с моделью.
    
