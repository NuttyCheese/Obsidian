**`NSManagedObjectModel`** — это класс из **[[Core Data]]**, представляющий **модель данных приложения**.

- Содержит **информацию обо всех сущностях (Entity)**, их атрибутах, связях и правилах валидации.
    
- Используется **[[NSPersistentStoreCoordinator]]** и **[[NSPersistentContainer]]** для загрузки и работы с хранилищем.
    
- Обычно создаётся автоматически из файла `.xcdatamodeld`, но можно создавать и программно.
    

Относится к: **Core Data → Model / [[Persistence]]**

---

## 🔹 Примеры кода

### 1. Загрузка модели из `.xcdatamodeld`

```swift
import CoreData

guard let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd"),
      let managedObjectModel = NSManagedObjectModel(contentsOf: modelURL) else {
    fatalError("Не удалось загрузить модель Core Data")
}
```

- `"Model"` — имя файла модели данных (`.xcdatamodeld`).
    
- `managedObjectModel` содержит все сущности и атрибуты.
    

---

### 2. Создание пустой модели вручную

```swift
let model = NSManagedObjectModel()

// Добавление сущности User
let userEntity = NSEntityDescription()
userEntity.name = "User"
userEntity.managedObjectClassName = "User"

let nameAttribute = NSAttributeDescription()
nameAttribute.name = "name"
nameAttribute.attributeType = .stringAttributeType
nameAttribute.isOptional = false

userEntity.properties = [nameAttribute]
model.entities = [userEntity]
```

- Создана модель с одной сущностью `User` и атрибутом `name`.
    

---

### 3. Использование модели с [[NSPersistentStoreCoordinator]]

```swift
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
try coordinator.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: storeURL)
```

- `NSManagedObjectModel` передаётся в **координатор**, который управляет хранилищем.
    

---

### 4. Использование с [[NSPersistentContainer]]

```swift
let container = NSPersistentContainer(name: "Model", managedObjectModel: managedObjectModel)
container.loadPersistentStores { storeDescription, error in
    if let error = error {
        fatalError("Ошибка загрузки Core Data: \(error)")
    }
}
```

- Позволяет интегрировать кастомную модель в стандартный стек Core Data.
    

---

### 5. Проверка сущностей в модели

```swift
for entity in managedObjectModel.entities {
    print("Сущность: \(entity.name ?? "нет имени")")
    for property in entity.properties {
        print(" - Атрибут/связь: \(property.name)")
    }
}
```

- Можно получить список всех **[[Entity]]** и их свойств.
    

---

### 🔹 Особенности

1. **NSManagedObjectModel = карта базы данных**: содержит сущности, атрибуты и связи.
    
2. Обычно загружается из `.xcdatamodeld` автоматически.
    
3. Можно создавать **динамически программно**, что полезно для тестирования или генерации моделей на лету.
    
4. Используется при **создании Persistent Store Coordinator** и **Persistent Container**.
    
