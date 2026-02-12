#save_data #swift #coredata 
## 📘 Определение

**Persistence в Core Data** — это термин, который описывает **способ постоянного хранения данных приложения**, чтобы они сохранялись между запусками.

- [[Core Data]] обеспечивает **слой абстракции над базой данных**, позволяя работать с объектами ([[NSManagedObject]]) без прямой работы с SQL.
    
- Включает **модель данных ([[NSManagedObjectModel]])**, **координатор хранилищ ([[NSPersistentStoreCoordinator]])** и **хранилища ([[NSPersistentStore]])**, например [[SQLite]], [[Binary]] или [[In-Memory]].
    
- Основная задача — **сохранение и извлечение данных из физического хранилища**.
    

Относится к: **Core Data → Persistence / Storage**

---

## 🔹 Примеры кода

### 1. Создание persistent контейнера для сохранения данных

```swift
import CoreData

let container = NSPersistentContainer(name: "Model")
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки persistent store: \(error)")
    }
    print("Persistent store загружен: \(storeDescription.url?.absoluteString ?? "")")
}
```

- `NSPersistentContainer` объединяет все уровни Core Data: модель, координатор, хранилище.
    

---

### 2. Сохранение объекта в persistent store

```swift
let context = container.viewContext
let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")
user.setValue(25, forKey: "age")

do {
    try context.save() // сохраняем данные в persistent store
    print("Данные сохранены")
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 3. Получение данных из persistent store

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

- Все объекты извлекаются из persistent store через **контекст**.
    

---

### 4. Использование фонового контекста для persistence

```swift
let bgContext = container.newBackgroundContext()
bgContext.perform {
    let newUser = User(context: bgContext)
    newUser.name = "Bob"
    newUser.age = 30
    
    try? bgContext.save() // сохранение в фоне
}
```

- Позволяет **не блокировать UI**, выполняя сохранение в фоне.
    

---

### 5. Проверка существования persistent store

```swift
if let store = container.persistentStoreCoordinator.persistentStores.first {
    print("Тип хранилища: \(store.type)")
    print("URL хранилища: \(store.url?.absoluteString ?? "In-memory")")
}
```

- Можно убедиться, что **данные сохраняются физически**, либо использовать in-memory хранилище для тестов.
    

---

### 🔹 Основные моменты Persistence в Core Data

| Компонент                        | Роль                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| [[NSManagedObjectModel]]         | Описание структуры данных (сущности, атрибуты, связи)        |
| [[NSPersistentStore]]            | Физическое хранилище ([[SQLite]], [[Binary]], [[In-Memory]]) |
| [[NSPersistentStoreCoordinator]] | Связывает модель с хранилищем                                |
| [[NSManagedObjectContext]]       | Контекст для создания/изменения/чтения объектов              |
| [[NSPersistentContainer]]        | Упрощает настройку всего стека Core Data                     |

- **Persistence** — это всё, что обеспечивает **сохранность данных приложения между запусками**.
    
- Core Data берет на себя **управление транзакциями, синхронизацией и масштабированием** данных.
    
