**In-Memory (в памяти)** — это способ хранения данных **не на диске, а в оперативной памяти (RAM)**.

- Данные существуют только во время работы приложения и исчезают после его закрытия.
    
- Используется для **тестов, временного кеша, быстрых операций с данными** или Core Data хранилища типа [[NSInMemoryStoreType]].
    
- Обеспечивает **максимальную скорость** чтения и записи, но не сохраняет данные между запусками.
    

Относится к: **Swift / [[Core Data]] → [[Persistence]] / Store**

---

## 🔹 Примеры кода

### 1. Создание In-Memory Core Data хранилища

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let model = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: model)

do {
    try coordinator.addPersistentStore(ofType: NSInMemoryStoreType, configurationName: nil, at: nil)
    print("In-Memory хранилище создано")
} catch {
    fatalError("Ошибка создания In-Memory хранилища: \(error)")
}
```

- `at: nil` указывает, что **нет физического файла**, всё хранится в памяти.
    

---

### 2. Использование In-Memory через [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model")
let description = NSPersistentStoreDescription()
description.type = NSInMemoryStoreType

container.persistentStoreDescriptions = [description]
container.loadPersistentStores { _, error in
    if let error = error {
        fatalError("Ошибка загрузки In-Memory store: \(error)")
    }
    print("In-Memory store загружен")
}
```

- Подходит для **unit-тестов**, когда не нужно сохранять реальные данные.
    

---

### 3. Сохранение объектов в In-Memory контексте

```swift
let context = container.viewContext
let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")

try? context.save() // изменения остаются только в памяти
```

- После завершения работы приложения данные **теряются**.
    

---

### 4. Тестирование Core Data с In-Memory

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

- Использование **In-Memory** идеально для юнит-тестов, не создавая файлов на диске.
    

---

### 5. Использование In-Memory для кеширования

```swift
var cache: [String: Any] = [:]

func saveToCache(key: String, value: Any) {
    cache[key] = value
}

func readFromCache(key: String) -> Any? {
    return cache[key]
}
```

- Простая **структура словаря** в памяти — тоже In-Memory хранение.
    

---

### 🔹 Особенности

|Свойство / Тип|Описание|
|---|---|
|`NSInMemoryStoreType`|Core Data: хранилище полностью в RAM|
|Скорость|Максимальная скорость операций|
|Сохранность|Данные теряются после закрытия приложения|
|Применение|Unit-тесты, временный кеш, прототипы|

- In-Memory идеально подходит для **временных данных** и **тестов**, но **не для постоянного хранения**.
    
