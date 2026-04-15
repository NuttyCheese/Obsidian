#foundation #nspointerarray #weak-reference #memory-management #collection #ios #swift

---

## NSPointerArray — Массив со слабыми ссылками

### Определение

**`NSPointerArray`** — это мутабельная коллекция из [[Foundation]], которая является аналогом [[NSMutableArray]], но с **гибкими настройками хранения указателей** (сильные, слабые, `copy`, `objectIdentifier`, `NULL`). В отличие от обычного [[Array]], `NSPointerArray` может хранить объекты через **слабые ссылки**, автоматически заменяя освобождённые объекты на `NULL` (но не удаляя их автоматически из массива).

Это идеальный инструмент для:
- **Списков делегатов (multicast delegates)** со слабыми ссылками
- **Наблюдателей (observers)**
- **Сценариев, где важен порядок элементов** (в отличие от [[NSHashTable]])
- **Хранения `NULL`-значений**

### Зачем это знать iOS-разработчику?

1.  **Слабые ссылки с сохранением порядка:** В отличие от `NSHashTable`, `NSPointerArray` сохраняет порядок элементов.
2.  **Multicast delegates:** Хранение нескольких делегатов без сильных ссылок.
3.  **Работа с `NULL`:** Может хранить пустые (null) указатели.
4.  **Кэширование:** Кэш, который не удерживает объекты, но сохраняет порядок.
5.  **Совместимость с [[Objective-C]]:** Работа с legacy-кодом и C-указателями.

---

### NSPointerArray vs Array vs NSHashTable

| Характеристика | `Array<Object>` | `NSHashTable` | `NSPointerArray` |
|---|---|---|---|
| **Тип ссылок** | Всегда сильные | Настраиваемые (сильные, слабые) | Настраиваемые (сильные, слабые) |
| **Уникальность элементов** | Нет | Да | Нет |
| **Порядок элементов** | Да | Нет | Да |
| **Автоудаление при деаллокации** | Нет | Да (из коллекции) | Нет (замена на `NULL`) |
| **Хранение `NULL`** | Нет | Нет | Да |
| **Generic поддержка** | Да | Нет | Нет |
| **Swift-совместимость** | Родной | Требуется приведение | Требуется приведение |

---

### Создание NSPointerArray

#### 1. **Слабые объекты (самый частый вариант)**

```swift
import Foundation

// Стандартный способ — слабые ссылки
let weakPointerArray = NSPointerArray.weakObjects()

// Или через инициализатор с опциями
let weakArray = NSPointerArray(options: .weakMemory)
```

#### 2. **Сильные объекты**

```swift
let strongPointerArray = NSPointerArray.strongObjects()
let strongArray = NSPointerArray(options: .strongMemory)
```

#### 3. **С копированием (NSCopying)**

```swift
let copyArray = NSPointerArray(options: .copyIn)
```

#### 4. **Пустой массив с возможностью хранения `NULL`**

```swift
let nullArray = NSPointerArray(options: .weakMemory)
nullArray.addPointer(nil)  // можно добавить NULL
```

---

### Доступные опции (NSPointerFunctionsOptions)

| Опция | Описание |
|---|---|
| **`.strongMemory`** | Сильные ссылки (по умолчанию). |
| **`.weakMemory`** | Слабые ссылки — объекты заменяются на `NULL` при деаллокации. |
| **`.copyIn`** | Копирование объектов перед вставкой (требует `NSCopying`). |
| **`.objectPointerPersonality`** | Сравнение по указателю, а не по содержимому. |
| **`.opaquePersonality`** | Для не-object указателей (void*). |
| **`.cStringPersonality`** | Для C-строк. |

---

### Базовые операции

#### Добавление и вставка

```swift
let pointerArray = NSPointerArray.weakObjects()

class MyClass {
    let id: Int
    init(id: Int) { self.id = id }
}

let obj1 = MyClass(id: 1)
let obj2 = MyClass(id: 2)

// Добавление в конец
pointerArray.addPointer(Unmanaged.passUnretained(obj1).toOpaque())
pointerArray.addPointer(Unmanaged.passUnretained(obj2).toOpaque())

// Вставка по индексу
let obj3 = MyClass(id: 3)
pointerArray.insertPointer(Unmanaged.passUnretained(obj3).toOpaque(), at: 1)
```

#### Удаление

```swift
// Удаление по индексу
pointerArray.removePointer(at: 0)

// Удаление всех элементов
pointerArray.removeAllPointers()
```

#### Доступ к элементам

```swift
// Получение указателя
if let pointer = pointerArray.pointer(at: 0) {
    let obj = Unmanaged<MyClass>.fromOpaque(pointer).takeUnretainedValue()
    print(obj.id)
}

// Компактный метод для извлечения объекта
if let obj = pointerArray.object(at: 0) as? MyClass {
    print(obj.id)
}
```

#### Количество элементов

```swift
let count = pointerArray.count
let allObjects = pointerArray.allObjects
```

---

### Главная фича: автоматическая замена на NULL при деаллокации

```swift
class TestObject {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) deallocated") }
}

let pointerArray = NSPointerArray.weakObjects()

var obj1: TestObject? = TestObject(name: "First")
var obj2: TestObject? = TestObject(name: "Second")

pointerArray.addPointer(Unmanaged.passUnretained(obj1!).toOpaque())
pointerArray.addPointer(Unmanaged.passUnretained(obj2!).toOpaque())

print(pointerArray.count)  // 2

obj1 = nil  // First deallocated
print(pointerArray.count)  // 2 (❗️ индекс остаётся)
print(pointerArray.object(at: 0) as? TestObject)  // nil (значение заменено на NULL)

// Удаление NULL-значений из массива
pointerArray.compact()
print(pointerArray.count)  // 1
```

---

### Очистка от NULL-значений (compact)

```swift
pointerArray.compact()  // Удаляет все NULL-указатели из массива
```

---

### Multicast Delegate (пример)

```swift
protocol DataLoaderDelegate: AnyObject {
    func dataLoaderDidFinish(data: String)
}

class DataLoader {
    // Слабые ссылки на всех делегатов (с сохранением порядка)
    private let delegates = NSPointerArray.weakObjects()
    
    func addDelegate(_ delegate: DataLoaderDelegate) {
        delegates.addPointer(Unmanaged.passUnretained(delegate).toOpaque())
    }
    
    func removeDelegate(_ delegate: DataLoaderDelegate) {
        for i in 0..<delegates.count {
            if let obj = delegates.object(at: i) as? DataLoaderDelegate,
               obj === delegate {
                delegates.removePointer(at: i)
                break
            }
        }
    }
    
    func notifyDelegates() {
        delegates.compact()  // Удаляем упавшие ссылки
        
        for i in 0..<delegates.count {
            if let delegate = delegates.object(at: i) as? DataLoaderDelegate {
                delegate.dataLoaderDidFinish(data: "Loaded data")
            }
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

### Удобное расширение для работы с объектами

```swift
extension NSPointerArray {
    func addObject(_ object: AnyObject?) {
        guard let object = object else {
            addPointer(nil)
            return
        }
        let pointer = Unmanaged.passUnretained(object).toOpaque()
        addPointer(pointer)
    }
    
    func insertObject(_ object: AnyObject?, at index: Int) {
        guard let object = object else {
            insertPointer(nil, at: index)
            return
        }
        let pointer = Unmanaged.passUnretained(object).toOpaque()
        insertPointer(pointer, at: index)
    }
    
    func object(at index: Int) -> AnyObject? {
        guard let pointer = pointer(at: index) else { return nil }
        return Unmanaged<AnyObject>.fromOpaque(pointer).takeUnretainedValue()
    }
    
    func compactAndRemoveNull() {
        compact()
    }
}

// Использование
let array = NSPointerArray.weakObjects()
array.addObject(obj1)
array.addObject(obj2)
let element = array.object(at: 0) as? MyClass
```

---

### NSPointerArray vs NSHashTable

| Характеристика | NSPointerArray | NSHashTable |
|---|---|---|
| **Уникальность элементов** | Нет | Да |
| **Порядок элементов** | Да | Нет |
| **Автоудаление при деаллокации** | Нет (замена на `NULL`) | Да (удаление из коллекции) |
| **Хранение `NULL`** | Да | Нет |
| **Производительность** | Высокая (для последовательного доступа) | Средняя |
| **Когда использовать** | Когда важен порядок | Когда важна уникальность |

---

### NSPointerArray vs NSMutableArray

| Характеристика | NSMutableArray | NSPointerArray |
|---|---|---|
| **Сильные ссылки** | ✅ (только сильные) | ✅ |
| **Слабые ссылки** | ❌ | ✅ |
| **Хранение `NULL`** | ❌ | ✅ |
| **Типобезопасность** | ❌ (id) | ❌ (void*) |
| **Производительность** | Высокая | Ниже (из-за обёрток) |

---

### Лучшие практики

1.  **Всегда вызывайте `.compact()` перед итерацией** — иначе получите `NULL`-значения.
2.  **Используйте расширения для удобной работы с объектами** — стандартный [[API]] неудобен.
3.  **Для хранения слабых ссылок без порядка используйте `NSHashTable`** — он автоматически удаляет объекты.
4.  **Не храните большие массивы со слабыми ссылками без очистки** — мёртвые `NULL`-значения копятся.
5.  **В Swift 5.9+ рассмотрите `WeakSet` / `WeakArray` на базе `NSHashTable` / `NSPointerArray`.**

---

### Альтернативы в чистом Swift

```swift
// Простая реализация WeakArray
struct Weak<T: AnyObject> {
    weak var value: T?
    init(_ value: T) { self.value = value }
}

class WeakArray<T: AnyObject> {
    private var items: [Weak<T>] = []
    
    func append(_ item: T) {
        items.append(Weak(item))
    }
    
    func compact() {
        items.removeAll { $0.value == nil }
    }
    
    var allObjects: [T] {
        compact()
        return items.compactMap { $0.value }
    }
}

// Использование
let weakArray = WeakArray<MyClass>()
weakArray.append(obj1)
weakArray.append(obj2)
weakArray.compact()
```

---

### Итог

**`NSPointerArray`** — это мощный инструмент для работы со слабыми ссылками в упорядоченных коллекциях. Он идеально подходит для:

1.  **Multicast delegates** — хранение нескольких делегатов с сохранением порядка.
2.  **Наблюдателей (observers)** — объекты заменяются на `NULL` при деаллокации.
3.  **Сценариев, где важен порядок элементов и нужны слабые ссылки.**
4.  **Хранения `NULL`-значений.**

Главные минусы — неавтоматическое удаление мёртвых элементов (нужно вызывать `compact()`) и отсутствие типобезопасности. В новых проектах часто предпочитают самодельные `WeakArray` на базе `NSPointerArray` или `NSHashTable` в зависимости от требований к уникальности и порядку.