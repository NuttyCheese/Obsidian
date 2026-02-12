#save_data #swift #coredata 
## 📘 Определение

**`NSManagedObject`** — базовый класс **[[Core Data]]**, представляющий **запись (объект) в базе данных**.

- Каждый объект Core Data, который хранится в [[NSPersistentStore]], **наследует `NSManagedObject`**.
    
- Позволяет работать с **атрибутами и связями сущностей** через ключи или сгенерированные свойства.
    
- В сочетании с `NSManagedObjectContext` объекты управляются, сохраняются и извлекаются из хранилища.
    

Относится к: **Core Data → [[Entity]] / [[Persistence]]**

---

## 🔹 Примеры кода

### 1. Создание объекта вручную

```swift
import CoreData

let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
let entity = NSEntityDescription.entity(forEntityName: "User", in: context)!
let user = NSManagedObject(entity: entity, insertInto: context)

user.setValue("Alice", forKey: "name")
user.setValue(25, forKey: "age")
```

---

### 2. Получение значения из `NSManagedObject`

```swift
let name = user.value(forKey: "name") as? String
let age = user.value(forKey: "age") as? Int
print("Имя: \(name ?? ""), возраст: \(age ?? 0)")
```

---

### 3. Сохранение изменений в контексте

```swift
do {
    try context.save()
    print("Объект User сохранён")
} catch {
    print("Ошибка сохранения: \(error)")
}
```

---

### 4. Fetch объектов

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")
do {
    let users = try context.fetch(fetchRequest)
    for u in users {
        print(u.value(forKey: "name") ?? "Нет имени")
    }
} catch {
    print("Ошибка fetch: \(error)")
}
```

---

### 5. Использование с кастомным классом

```swift
@objc(User)
class User: NSManagedObject {
    @NSManaged var name: String
    @NSManaged var age: Int16
}

// Создание и сохранение
let newUser = User(context: context)
newUser.name = "Bob"
newUser.age = 30

try? context.save()
```

- Использование **сгенерированных свойств** делает код **безопаснее и читабельнее**, чем через `setValue`/`value(forKey:)`.
    

---

### 🔹 Особенности

1. Все свойства `NSManagedObject` должны быть помечены как **`@NSManaged`**.
    
2. Объекты управляются **контекстом**, нельзя использовать вне него без правильного сохранения/мержа.
    
3. Связи между сущностями (`to-one`, `to-many`) также работают через `NSManagedObject`.
    
