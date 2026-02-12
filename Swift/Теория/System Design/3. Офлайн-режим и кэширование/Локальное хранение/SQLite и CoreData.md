#system_design
## Определение

### SQLite

**SQLite** — это встроенная легковесная реляционная база данных.

- Данные хранятся в виде таблиц с колонками и строками.
    
- Все хранится локально в одном файле на устройстве.
    
- Используется напрямую через SQL-запросы или через обертки.
    

### Core Data

**[[Core Data]]** — это **фреймворк для управления объектами**, предоставляемый Apple.

- Позволяет хранить объекты [[Swift]]/[[Objective-C]] в базе данных.
    
- Может использовать [[SQLite]] как внутренний механизм хранения, но работает через **объектно-ориентированную модель**.
    

> [[Realm]], Core Data и SQLite — все три решения для локального хранения, но с разными уровнями абстракции.

---

## Основные отличия Realm, Core Data и SQLite

|Особенность|SQLite|Core Data|Realm|
|---|---|---|---|
|Уровень абстракции|Низкий (SQL)|Средний (объекты + SQL под капотом)|Высокий (объекты)|
|Прямой SQL|Да|Нет (только через NSPredicate/FetchRequest)|Нет|
|Скорость|Быстро, зависит от SQL|Средняя|Очень высокая|
|Поддержка оффлайн|Да|Да|Да|
|Реальное время|Нет|Есть уведомления|Есть уведомления|
|Сложность|Нужно писать SQL|Модели, контексты|Модели, простые транзакции|
|Мультиплатформа|Да|iOS/macOS|iOS/Android/cross-platform|

---

## SQLite в iOS

### Прямое использование

```swift
import SQLite3

var db: OpaquePointer?
if sqlite3_open("\(NSHomeDirectory())/Documents/test.db", &db) == SQLITE_OK {
    print("База открыта")
}

// Создание таблицы
let createTableQuery = "CREATE TABLE IF NOT EXISTS Users (id TEXT PRIMARY KEY, name TEXT)"
sqlite3_exec(db, createTableQuery, nil, nil, nil)

// Вставка данных
let insertQuery = "INSERT INTO Users (id, name) VALUES ('123', 'Alex')"
sqlite3_exec(db, insertQuery, nil, nil, nil)
```

### Плюсы SQLite

- Контроль над SQL-запросами.
    
- Легковесность и стабильность.
    
- Прямой доступ к реляционным данным.
    

### Минусы SQLite

- Нужно писать SQL вручную.
    
- Нет удобной интеграции с объектами Swift.
    
- Меньше поддержки оффлайн уведомлений и связей объектов.
    

---

## Core Data в iOS

### Основные компоненты

1. **Managed Object Model (NSManagedObjectModel)** — схема данных.
    
2. **Managed Object Context (NSManagedObjectContext)** — контекст для чтения/записи объектов.
    
3. **Persistent Store Coordinator (NSPersistentStoreCoordinator)** — управляет хранением данных (SQLite, Binary, In-Memory).
    

### Пример модели

```swift
import CoreData

class User: NSManagedObject {
    @NSManaged var id: String
    @NSManaged var name: String
}
```

### Добавление объекта

```swift
let context = persistentContainer.viewContext
let user = User(context: context)
user.id = "123"
user.name = "Alex"

try? context.save()
```

### Получение объектов

```swift
let fetchRequest: NSFetchRequest<User> = User.fetchRequest()
let users = try? context.fetch(fetchRequest)
```

### Плюсы Core Data

- Интеграция с объектно-ориентированным Swift кодом.
    
- Автоматические уведомления об изменениях (полезно для UI).
    
- Поддержка сложных отношений и фильтров через `NSPredicate`.
    
- Использует SQLite под капотом (или Binary/In-Memory).
    

### Минусы Core Data

- Сложнее Realm в освоении.
    
- Требует управления контекстами и транзакциями.
    
- Иногда непредсказуемое поведение при больших данных.
    

---

## Сравнение Realm и Core Data

|Критерий|Core Data|Realm|
|---|---|---|
|Абстракция|Средняя|Высокая|
|Реальное время|Да|Да|
|Скорость|Средняя|Высокая|
|Оффлайн|Да|Да|
|Сложность кода|Средняя|Простая|
|Мультиплатформа|Нет|Да|

---

## Рекомендации для iOS

- **SQLite**: когда нужен полный контроль над SQL и сложные запросы.
    
- **Core Data**: для стандартных приложений iOS с объектно-ориентированным подходом, связями и уведомлениями UI.
    
- **Realm**: для высокой производительности, простого синтаксиса и кроссплатформенных проектов.
    

---

## Итог

- **SQLite** — низкоуровневая реляционная база.
    
- **Core Data** — фреймворк Apple для управления объектами, использующий SQLite под капотом.
    
- **Realm** — современное объектно-ориентированное решение с высокой производительностью и оффлайн поддержкой.
    
- Выбор зависит от: **объема данных, требуемой скорости, сложности моделей и кроссплатформенности**.
    

---
