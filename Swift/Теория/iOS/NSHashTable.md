#foundation #nshashtable #weak-reference #memory-management #collection #ios #swift`

---
## NSHashTable — Коллекция со слабыми ссылками

### Определение

**`NSHashTable`** — это мутабельная коллекция из Foundation, которая является аналогом `NSSet`, но с **гибкими настройками хранения ссылок** (сильные, слабые, `copy`, `objectIdentifier`). В отличие от обычного [[Set Collection|Set]], `NSHashTable` может хранить объекты через **слабые ссылки**, автоматически удаляя элементы, когда на них больше нет сильных ссылок в памяти.

Это идеальный инструмент для:
- **Списков делегатов (multicast delegates)**
- **Наблюдателей (observers)**
- **Кэшей, не препятствующих освобождению объектов**
- **Сценариев, где нужно избежать retain cycles**

### Зачем это знать iOS-разработчику?

1.  **Слабые ссылки на объекты:** `NSHashTable` может автоматически удалять объекты, когда они деаллоцируются.
2.  **Multicast delegates:** Хранение нескольких делегатов без сильных ссылок.
3.  **Кэширование:** Кэш, который не удерживает объекты (аналог [[NSPointerArray]], но для уникальности).
4.  **Наблюдатели (Observers):** Реализация паттерна Observer без утечек памяти.
5.  **Совместимость с [[Objective-C]]:** Работа с legacy-кодом.

---

### NSHashTable vs Set

| Характеристика                   | `Set<Object>`                 | `NSHashTable`                                     |
| -------------------------------- | ----------------------------- | ------------------------------------------------- |
| **Тип ссылок**                   | Всегда сильные                | Настраиваемые (сильные, слабые, копии, указатели) |
| **Автоудаление при деаллокации** | Нет                           | Да (при использовании слабых ссылок)              |
| **Потокобезопасность**           | Нет (требуется синхронизация) | Опционально (`.weakObjects()` потокобезопасен)    |
| **Generic поддержка**            | Да                            | Нет (только [[AnyObject]])                        |
| **Производительность**           | Высокая                       | Ниже (из-за обёрток)                              |
| **Swift-совместимость**          | Родной                        | Требуется приведение типов                        |

---

### Создание NSHashTable

#### 1. **Слабые объекты (самый частый вариант)**

```swift
import Foundation

// Для хранения слабых ссылок на объекты
let weakHashTable = NSHashTable<AnyObject>.weakObjects()

// Или через инициализатор
let weakTable = NSHashTable(options: .weakMemory)
```

#### 2. **Сильные объекты**

```swift
let strongHashTable = NSHashTable<AnyObject>.strongObjects()
let strongTable = NSHashTable(options: .strongMemory)
```

#### 3. **С копированием (NSCopying)**

```swift
let copyTable = NSHashTable(options: .copyIn)
```

#### 4. **По идентификатору объекта (ObjectIdentifier)**

```swift
let objectIdTable = NSHashTable(options: .objectPointerPersonality)
```

---

### Доступные опции (NSHashTableOptions)

| Опция | Описание |
|---|---|
| **`.strongMemory`** | Сильные ссылки (по умолчанию для `NSHashTable`). |
| **`.weakMemory`** | Слабые ссылки — объекты автоматически удаляются при деаллокации. |
| **`.copyIn`** | Копирование объектов перед вставкой (требует `NSCopying`). |
| **`.objectPointerPersonality`** | Сравнение по указателю, а не по содержимому. |
| **`.elementEqualFunction`** | Кастомное сравнение элементов. |
| **`.elementHashFunction`** | Кастомная хеш-функция. |

---

### Базовые операции

#### Добавление и удаление

```swift
let hashTable = NSHashTable<AnyObject>.weakObjects()

class MyClass {
    let id: Int
    init(id: Int) { self.id = id }
}

let obj1 = MyClass(id: 1)
let obj2 = MyClass(id: 2)

hashTable.add(obj1)
hashTable.add(obj2)

print(hashTable.count)  // 2

hashTable.remove(obj1)
print(hashTable.count)  // 1
```

#### Проверка наличия

```swift
let contains = hashTable.contains(obj2)
```

#### Итерация

```swift
for object in hashTable.allObjects {
    if let obj = object as? MyClass {
        print(obj.id)
    }
}

// Или с использованием enumerator
let enumerator = hashTable.objectEnumerator()
while let object = enumerator.nextObject() as? MyClass {
    print(object.id)
}
```

#### Очистка коллекции

```swift
hashTable.removeAllObjects()
```

---

### Главная фича: автоматическое удаление при деаллокации

```swift
class TestObject {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

let hashTable = NSHashTable<AnyObject>.weakObjects()

var obj1: TestObject? = TestObject(name: "First")
var obj2: TestObject? = TestObject(name: "Second")

hashTable.add(obj1!)
hashTable.add(obj2!)
print(hashTable.count)  // 2

obj1 = nil  // First deallocated
print(hashTable.count)  // 1 (obj1 автоматически удалён)

obj2 = nil  // Second deallocated
print(hashTable.count)  // 0
```

---

### Multicast Delegate (пример)

```swift
protocol DataLoaderDelegate: AnyObject {
    func dataLoaderDidFinish(data: String)
}

class DataLoader {
    // Слабые ссылки на всех делегатов
    private let delegates = NSHashTable<AnyObject>.weakObjects()
    
    func addDelegate(_ delegate: DataLoaderDelegate) {
        delegates.add(delegate)
    }
    
    func removeDelegate(_ delegate: DataLoaderDelegate) {
        delegates.remove(delegate)
    }
    
    func notifyDelegates() {
        for case let delegate as DataLoaderDelegate in delegates.allObjects {
            delegate.dataLoaderDidFinish(data: "Loaded data")
        }
    }
}

class ViewController: UIViewController, DataLoaderDelegate {
    func dataLoaderDidFinish(data: String) {
        print("Received: \(data)")
    }
    
    func test() {
        let loader = DataLoader()
        loader.addDelegate(self)
        loader.notifyDelegates()  // Received: Loaded data
    }
}
```

---

### Observer Pattern (пример)

```swift
protocol SettingsObserver: AnyObject {
    func themeDidChange()
}

class SettingsManager {
    private let observers = NSHashTable<AnyObject>.weakObjects()
    
    func addObserver(_ observer: SettingsObserver) {
        observers.add(observer)
    }
    
    func removeObserver(_ observer: SettingsObserver) {
        observers.remove(observer)
    }
    
    func setTheme() {
        for case let observer as SettingsObserver in observers.allObjects {
            observer.themeDidChange()
        }
    }
}
```

---

### NSHashTable vs Set (производительность)

```swift
import Foundation

class TestObject {
    let id: Int
    init(id: Int) { self.id = id }
}

// Set
var set = Set<TestObject>()
for i in 0..<10000 {
    set.insert(TestObject(id: i))
}

// NSHashTable (weak)
let hashTable = NSHashTable<AnyObject>.weakObjects()
for i in 0..<10000 {
    hashTable.add(TestObject(id: i))
}
```

**Примечание:** `Set` работает быстрее, но не поддерживает слабые ссылки. Выбор зависит от задачи.

---

### NSHashTable vs NSPointerArray

| Характеристика             | NSHashTable                      | NSPointerArray                          |
| -------------------------- | -------------------------------- | --------------------------------------- |
| **Уникальность элементов** | Да (как [[Set Collection\|Set]]) | Нет (как [[Array]])                     |
| **Порядок элементов**      | Не гарантирован                  | Гарантирован                            |
| **Слабые ссылки**          | Да                               | Да                                      |
| **Производительность**     | Средняя                          | Высокая (для последовательного доступа) |
| **Использование**          | Уникальные элементы              | Упорядоченные элементы                  |

---

### Лучшие практики

1.  **Используйте `.weakObjects()` для списков делегатов и наблюдателей** — это предотвращает утечки памяти.
2.  **Для хранения сильных ссылок используйте обычный `Set`** — он производительнее и типобезопаснее.
3.  **При работе с `NSHashTable` всегда приводите типы** — `as?` или `as!`.
4.  **Не рассчитывайте на порядок элементов** — `NSHashTable` не гарантирует его.
5.  **Очищайте коллекцию в [[deinit]]** — хотя слабые ссылки очищаются автоматически, для сильных ссылок это важно.

```swift
deinit {
    hashTable.removeAllObjects()
}
```

---

### Итог

**`NSHashTable`** — это мощный инструмент для работы со слабыми ссылками в коллекциях. Он идеально подходит для:

1.  **Multicast delegates** — хранение нескольких делегатов без утечек.
2.  **Наблюдателей (observers)** — автоматическое удаление умерших объектов.
3.  **Кэшей** — не мешает освобождению памяти.
4.  **Сценариев, где нужна коллекция уникальных объектов со слабыми ссылками.**

Главный минус — отсутствие типобезопасности и более низкая производительность по сравнению с нативным `Set`. В новых проектах, где нужны слабые ссылки, часто предпочитают `NSHashTable` или самодельные обёртки на базе `NSHashTable`.