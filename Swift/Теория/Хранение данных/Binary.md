**Binary хранение данных в [[Swift]]** — это способ **сериализации и сохранения данных в бинарном формате** на диск или в память.

- Данные сохраняются не как текст (например, [[JSON]] или [[XML]]), а в **компактном двоичном виде**, что экономит место и повышает скорость чтения/записи.
    
- Часто используется для **[[Core Data]] ([[NSBinaryStoreType]])**, **архивирования объектов (`NSKeyedArchiver`)**, **кеширования**, или передачи данных по сети.
    
- В отличие от текстовых форматов, бинарное хранение менее читаемо человеком, но быстрее и эффективнее для больших объёмов данных.
    

Относится к: **Swift → [[Foundation]] / [[Persistence]] / Core Data**

---

## 🔹 Примеры кода

### 1. Сохранение словаря в бинарный файл через `NSKeyedArchiver`

```swift
import Foundation

let dataDict: [String: Any] = ["name": "Alice", "age": 25]
let fileURL = FileManager.default.temporaryDirectory.appendingPathComponent("data.bin")

do {
    let data = try NSKeyedArchiver.archivedData(withRootObject: dataDict, requiringSecureCoding: false)
    try data.write(to: fileURL)
    print("Данные сохранены в бинарном файле")
} catch {
    print("Ошибка записи: \(error)")
}
```

---

### 2. Чтение бинарного файла через `NSKeyedUnarchiver`

```swift
do {
    let data = try Data(contentsOf: fileURL)
    if let loadedDict = try NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(data) as? [String: Any] {
        print("Загруженные данные: \(loadedDict)")
    }
} catch {
    print("Ошибка чтения: \(error)")
}
```

---

### 3. Binary-хранилище в Core Data (`NSBinaryStoreType`)

```swift
import CoreData

let modelURL = Bundle.main.url(forResource: "Model", withExtension: "momd")!
let model = NSManagedObjectModel(contentsOf: modelURL)!
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: model)

let storeURL = FileManager.default.temporaryDirectory.appendingPathComponent("store.data")
try coordinator.addPersistentStore(ofType: NSBinaryStoreType, configurationName: nil, at: storeURL)
```

- Вместо [[SQLite]] используется **двойной формат** для всех сущностей.
    
- Подходит для **малых и средних объёмов данных**, но не масштабируется как SQLite.
    

---

### 4. Сериализация кастомного объекта с [[Codable]] в бинарный формат

```swift
struct User: Codable {
    let name: String
    let age: Int
}

let user = User(name: "Bob", age: 30)
let fileURL = FileManager.default.temporaryDirectory.appendingPathComponent("user.bin")

do {
    let data = try PropertyListEncoder().encode(user)
    try data.write(to: fileURL)
} catch {
    print("Ошибка записи: \(error)")
}

// Чтение
do {
    let data = try Data(contentsOf: fileURL)
    let loadedUser = try PropertyListDecoder().decode(User.self, from: data)
    print("Загруженный пользователь: \(loadedUser)")
} catch {
    print("Ошибка чтения: \(error)")
}
```

- Использует **PropertyListEncoder/Decoder**, который по умолчанию создаёт **binary plist**.
    

---

### 5. Кэширование изображений в бинарном формате

```swift
if let image = UIImage(named: "avatar") {
    let data = image.pngData() // или jpegData(compressionQuality: 0.8)
    let fileURL = FileManager.default.temporaryDirectory.appendingPathComponent("avatar.bin")
    try? data?.write(to: fileURL)
}
```

- Быстрое сохранение больших объектов (например, фото) в бинарном виде.
    

---

### 🔹 Особенности Binary-хранения

| Метод / Тип           | Описание                                                     |
| --------------------- | ------------------------------------------------------------ |
| `NSKeyedArchiver`     | Архивация объектов [[Objective-C]] / Swift в бинарный формат |
| `PropertyListEncoder` | Кодирование [[Codable]] объектов в бинарный plist            |
| `NSBinaryStoreType`   | Core Data: хранение сущностей в бинарном формате             |
| `Data.write(to:)`     | Запись бинарного потока на диск                              |
| Быстрота              | Чтение/запись быстрее текстовых форматов                     |
| Читаемость            | Данные **не читаемы человеком**                              |

- **Binary хранение** идеально для **кеширования, Core Data in-memory/файлового хранилища и архивирования объектов**.
    
- Для больших, часто изменяемых данных лучше использовать **SQLite** вместо бинарного файла.
    
