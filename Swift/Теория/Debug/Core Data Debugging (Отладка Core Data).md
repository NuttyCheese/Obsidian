![Image](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/Art/Model_Editor_2x.png)

![Image](https://useyourloaf.com/blog/debugging-core-data/cover.png)

![Image](https://www.advancedswift.com/content/images/2021/07/AdvancedSwift_EntityInspector_ParentEntity.png)

![Image](https://i.sstatic.net/Y8Xxi.png)

## Что такое Core Data Debugging

**Core Data Debugging** — это анализ:

- корректности сохранения данных
    
- выполнения fetch-запросов
    
- работы контекстов ([[NSManagedObjectContext]])
    
- потоков
    
- производительности
    
- конфликтов сохранения
    

Проще говоря:

> Это контроль жизненного цикла данных в persistent store.

[[Core Data]] — мощный инструмент, но ошибки в нём часто неочевидны.

---

# 🧱 Основные направления отладки

---

## 1️⃣ Проверка сохранения (Save)

### Симптом:

- данные "не сохраняются"
    
- после перезапуска их нет
    

### Проверка:

```swift
do {
    try context.save()
} catch {
    print(error)
}
```

Важно:

- всегда проверять `hasChanges`
    
- всегда логировать error
    

---

## 2️⃣ Включение SQL Debug

Можно включить флаг запуска:

```
-com.apple.CoreData.SQLDebug 1
```

[[Xcode]] → Scheme → Arguments → Arguments Passed On Launch.

После этого в консоли будет видно:

```sql
SELECT ...
INSERT ...
UPDATE ...
```

Это позволяет:

- увидеть реальные SQL-запросы
    
- понять, выполняется ли fetch
    
- проверить индексы
    
- увидеть N+1 проблему
    

---

## 3️⃣ Проверка Fetch Request

Если данные не находятся:

Проверить:

- entityName
    
- predicate
    
- sortDescriptors
    
- fetchLimit
    

Пример:

```swift
let request: NSFetchRequest<User> = User.fetchRequest()
request.predicate = NSPredicate(format: "id == %@", id)
```

Частая ошибка — неверный тип в predicate.

---

## 4️⃣ Проблемы с контекстами

Core Data работает через:

- main context
    
- background context
    

Если сохраняешь в background, но не merge’ишь в main — UI не обновится.

Проверка:

- какой context используется
    
- вызывается ли `perform`
    
- mergePolicy
    

---

# 🧵 Проблемы с потоками (очень частые)

Core Data НЕ thread-safe.

Ошибка:

```
NSManagedObjectContext cannot be used across threads
```

Правильно:

```swift
context.perform {
    // работа с объектами
}
```

Нельзя передавать [[NSManagedObject]] между потоками.  
Нужно передавать objectID.

---

# 🧪 Практические сценарии

---

## 📦 1. Данные дублируются

Причина:

- каждый раз создаётся новый объект
    
- нет проверки существования
    

Решение:

- делать fetch перед insert
    
- использовать уникальный constraint в модели
    

---

## 📉 2. Медленный список

Проверить:

- fetchBatchSize
    
- includesPropertyValues
    
- prefetching
    

Если видишь в SQL Debug десятки SELECT — есть проблема.

---

## 🔄 3. Конфликт сохранения

Ошибка:

```
Could not merge changes
```

Нужно настроить:

```swift
context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
```

Выбор merge policy критичен.

---

# 🚀 Продвинутая отладка

---

## 🔥 1. Instruments → Core Data Template

Позволяет анализировать:

- количество fetch
    
- faulting
    
- cache misses
    
- сохранения
    

Очень полезно при сложной модели.

---

## 🔥 2. Faulting

Core Data использует [[lazy]]-загрузку.

Если видишь много fault firing — возможна проблема в архитектуре.

---

## 🔥 3. Проверка persistent store

Можно проверить:

- путь к [[SQLite]]
    
- удаляется ли база
    
- создаётся ли новая
    

Иногда баг просто в том, что база пересоздаётся.

---

# 📊 Частые ошибки

|Ошибка|Причина|
|---|---|
|Данные не сохраняются|save не вызван|
|UI не обновляется|не тот context|
|Краш на другом потоке|объект передан между потоками|
|Дубли|нет уникальных ограничений|
|Медленно|нет batchSize|

---

# 🧠 Правильный алгоритм отладки Core Data

1. Проверить save (try/catch)
    
2. Включить SQLDebug
    
3. Проверить fetch
    
4. Проверить контексты
    
5. Проверить потоки
    
6. Проверить merge policy
    
7. Проверить производительность через Instruments
    

---

# 🎯 Важно понимать

Core Data — это не просто база данных.  
Это:

- object graph manager
    
- система кэширования
    
- ORM
    
- механизм синхронизации контекстов
    

Ошибки чаще архитектурные, чем синтаксические.

---

# 💡 Практический совет

Если проект большой:

- Разделять main и background context
    
- Всегда использовать objectID для передачи
    
- Включать SQLDebug при сложных багах
    
- Делать уникальные constraints в модели
    
- Следить за mergePolicy
    

---
