**`NSPersistentStoreCoordinator`** — это класс из **[[Core Data]]**, который **связывает модель данных (`NSManagedObjectModel`) с одним или несколькими физическими хранилищами (`NSPersistentStore`)**.

- Координатор управляет всеми хранилищами приложения и обеспечивает **синхронизацию данных** между ними.
    
- Он отвечает за **чтение и запись объектов** из/в физические файлы или in-memory хранилища.
    
- Контексты ([[NSManagedObjectContext]]) используют координатор для выполнения операций с данными.
    

Относится к: **Core Data → [[Persistence]] / [[Stack]]**

---

## 🔹 Примеры кода

### 1. Создание координатора с моделью данных

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let managedObjectModel = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
```

- Координатор связывает модель с будущими хранилищами.
    

---

### 2. Добавление [[SQLite]] хранилища

```swift
let storeURL = URL(fileURLWithPath: "/path/to/store.sqlite")
do {
    let store = try coordinator.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: storeURL)
    print("Хранилище добавлено: \(store)")
} catch {
    fatalError("Не удалось добавить хранилище: \(error)")
}
```

- Хранилище теперь готово к работе с объектами Core Data.
    

---

### 3. Добавление [[In-Memory]] хранилища (для тестов)

```swift
do {
    let store = try coordinator.addPersistentStore(ofType: NSInMemoryStoreType, configurationName: nil, at: nil)
    print("In-memory хранилище добавлено: \(store)")
} catch {
    fatalError("Ошибка добавления in-memory хранилища: \(error)")
}
```

- Не сохраняется на диск, удобно для юнит-тестов.
    

---

### 4. Связывание контекста с координатором

```swift
let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
context.persistentStoreCoordinator = coordinator

// Теперь все fetch и save будут использовать координатор
```

- Любой контекст должен иметь **связанный координатор**, чтобы читать/писать объекты.
    

---

### 5. Получение списка всех хранилищ

```swift
for store in coordinator.persistentStores {
    print("Store type: \(store.type), URL: \(store.url?.absoluteString ?? "In-memory")")
}
```

- Можно работать с **несколькими хранилищами одновременно**.
    

---

### 🔹 Основные моменты

|Свойство / Метод|Назначение|
|---|---|
|`addPersistentStore(ofType:configurationName:at:options:)`|Добавление нового хранилища|
|`remove(_:)`|Удаление хранилища из координатора|
|`persistentStores`|Список всех подключённых хранилищ|
|`managedObjectModel`|Модель данных, связанная с координатором|
|`destroyPersistentStore(at:ofType:options:)`|Удаление физического хранилища с диска|

- **NSPersistentStoreCoordinator** — это «связующее звено» между **моделью данных** и **физическим хранилищем**.
    
- Обычно используется **автоматически через [[NSPersistentContainer]]**, но знание координатора важно для **кастомной настройки Core Data**.
    
