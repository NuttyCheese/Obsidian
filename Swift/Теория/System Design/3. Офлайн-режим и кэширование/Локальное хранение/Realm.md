#system_design
## Определение

**Realm** — это современная база данных для мобильных приложений, позволяющая хранить данные **локально на устройстве** и работать с ними **очень быстро и просто**, без использования [[Core Data]] или [[SQLite]] напрямую.

> Realm оптимизирован для работы в [[iOS]], Android и кроссплатформенных приложениях.

---

## Основные особенности Realm

1. **Локальная база данных**
    
    - Данные хранятся на устройстве, не требует сетевого соединения.
        
2. **Объектно-ориентированная модель**
    
    - Модели данных описываются как обычные классы Swift с наследованием от `Object`.
        
3. **Реальное время и уведомления**
    
    - Можно подписываться на изменения данных и обновлять UI автоматически.
        
4. **Высокая производительность**
    
    - Быстрее Core Data и SQLite при работе с большими объёмами данных.
        
5. **Поддержка оффлайн-режима**
    
    - Данные доступны даже без сети.
        
    - Можно синхронизировать с сервером при подключении (Realm Sync, платная функция).
        

---

## Установка Realm

Через Swift Package Manager:

```swift
dependencies: [
    .package(url: "https://github.com/realm/realm-cocoa.git", from: "10.0.0")
]
```

или через CocoaPods:

```ruby
pod 'RealmSwift'
```

---

## Основные компоненты

1. **Object**
    
    - Класс модели данных.
        
    - Пример:
        

```swift
import RealmSwift

class User: Object {
    @objc dynamic var id: String = ""
    @objc dynamic var name: String = ""
    @objc dynamic var email: String = ""

    override static func primaryKey() -> String? {
        return "id"
    }
}
```

2. **Realm**
    
    - Основной объект для доступа к базе.
        
    - Пример:
        

```swift
let realm = try! Realm()
```

3. **Transactions**
    
    - Все изменения должны быть внутри транзакций:
        

```swift
try! realm.write {
    realm.add(user)
}
```

---

## Основные операции

### 1. Добавление данных

```swift
let user = User()
user.id = "123"
user.name = "Alex"
user.email = "alex@example.com"

try! realm.write {
    realm.add(user)
}
```

### 2. Получение данных

```swift
let users = realm.objects(User.self) // все объекты User
let user = realm.object(ofType: User.self, forPrimaryKey: "123")
```

### 3. Обновление данных

```swift
if let user = realm.object(ofType: User.self, forPrimaryKey: "123") {
    try! realm.write {
        user.name = "Alexander"
    }
}
```

### 4. Удаление данных

```swift
if let user = realm.object(ofType: User.self, forPrimaryKey: "123") {
    try! realm.write {
        realm.delete(user)
    }
}
```

---

## Подписка на изменения (Real-time updates)

```swift
let users = realm.objects(User.self)
let token = users.observe { changes in
    switch changes {
    case .initial:
        print("Initial data: \(users)")
    case .update(_, let deletions, let insertions, let modifications):
        print("Deleted: \(deletions), Inserted: \(insertions), Modified: \(modifications)")
    case .error(let error):
        print("Error: \(error)")
    }
}
```

- Удобно для **обновления UI автоматически** при изменении данных.
    

---

## Применение Realm в оффлайн-режиме

1. Хранение данных API-запросов локально для отображения при отсутствии сети.
    
2. Кэширование списков, изображений (ссылка на локальный файл) и истории пользователя.
    
3. Поддержка **синхронизации с сервером** через Realm Sync (опционально).
    

---

## Преимущества Realm для iOS

- Простая модель данных и синтаксис Swift.
    
- Высокая скорость записи и чтения данных.
    
- Автоматические уведомления об изменениях.
    
- Подходит для оффлайн-приложений и кэширования.
    
- Легко интегрируется с другими слоями приложения (UI, сетевой слой).
    

---

## Итог

- **Realm** — мощное решение для локального хранения данных в iOS.
    
- Работает быстро, просто, поддерживает **реальное время** и оффлайн.
    
- Подходит для кэширования API, истории действий пользователя и любых локальных моделей.
    
- Отлично интегрируется с современными архитектурами (MVVM, Clean Swift).
    

---
