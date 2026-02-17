**Entity в [[Core Data]]** — это абстракция, представляющая **сущность данных (таблицу)** в модели данных (`.xcdatamodeld`).

- Каждая сущность содержит **атрибуты** (колонки) и **связи** (relationships) с другими сущностями.
    
- Entity используется как **шаблон** для создания объектов **[[NSManagedObject]]**.
    
- Сущности могут быть связаны **“один к одному”, “один ко многим” и “многие ко многим”**.
    

Относится к: **Core Data → Model / NSManagedObject**

---

## 🔹 Примеры кода

### 1. Создание сущности в Core Data модель редакторе

```text
Entity: User
Attributes:
- name: String
- age: Int16
```

- В Xcode создаём `.xcdatamodeld`, добавляем **Entity User** с атрибутами `name` и `age`.
    

---

### 2. Создание объекта сущности через код

```swift
import CoreData

let user = NSEntityDescription.insertNewObject(forEntityName: "User", into: context)
user.setValue("Alice", forKey: "name")
user.setValue(25, forKey: "age")
```

- `User` — имя **Entity**, используется для создания `NSManagedObject`.
    

---

### 3. Fetch объектов сущности

```swift
let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "User")

do {
    let users = try context.fetch(fetchRequest)
    users.forEach { print($0.value(forKey: "name") ?? "") }
} catch {
    print("Ошибка fetch: \(error)")
}
```

- Получаем все объекты типа `User` из Core Data.
    

---

### 4. Кастомный класс для Entity

```swift
@objc(User)
class User: NSManagedObject {
    @NSManaged var name: String
    @NSManaged var age: Int16
}

// Использование
let newUser = User(context: context)
newUser.name = "Bob"
newUser.age = 30
try? context.save()
```

- Позволяет работать с **типизированными свойствами**, а не через `setValue` / `value(forKey:)`.
    

---

### 5. Связи между сущностями

```text
Entity: User
Attributes: name, age
Relationship: pets -> Pet (to-many)

Entity: Pet
Attributes: name
Relationship: owner -> User (to-one)
```

- Связи помогают моделировать **более сложные структуры данных**.
    
- В коде можно обращаться через свойства:
    

```swift
let pet = Pet(context: context)
pet.name = "Rex"
pet.owner = newUser
newUser.addToPets(pet)
try? context.save()
```

---

### 🔹 Особенности

1. **Entity = таблица**, атрибут = колонка, объект = запись.
    
2. Связи могут быть **один к одному, один ко многим, многие ко многим**.
    
3. Использование кастомных классов повышает **безопасность типов**.
    
4. Core Data генерирует **NSManagedObject subclasses**, которые соответствуют сущностям.
    
