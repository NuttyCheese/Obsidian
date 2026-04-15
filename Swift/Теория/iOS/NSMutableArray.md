#foundation #nsmutablearray #array #collection #objective-c #ios #swift

---
## NSMutableArray — Мутабельный массив из Foundation

### Определение

**`NSMutableArray`** — это класс из [[Foundation]] (наследник [[NSArray]]), представляющий **упорядоченную мутабельную коллекцию** объектов. В отличие от `NSArray` (иммутабельный), `NSMutableArray` позволяет добавлять, удалять и заменять элементы после создания.

В современной Swift-разработке `NSMutableArray` почти полностью вытеснен нативным `Array`, но остаётся востребованным при:
- **Работе с legacy [[Objective-C]] кодом**
- **Взаимодействии с [[API]], требующими `NSMutableArray`**
- **Низкоуровневых операциях с Objective-C runtime**
- **Использовании в AppKit (macOS) или старых [[UIKit]]-проектах**

### Зачем это знать iOS-разработчику?

1.  **Совместимость с Objective-C:** Многие старые библиотеки и API ожидают `NSMutableArray`.
2.  **Интеграция с [[Core Data]]:** Некоторые отношения «многие ко многим» возвращают `NSMutableArray`.
3.  **Понимание Foundation:** Глубокое знание Foundation помогает отлаживать и оптимизировать код.
4.  **Миграция легаси:** При переносе старых проектов на Swift нужно понимать `NSMutableArray`.
5.  **Производительность:** В некоторых сценариях `NSMutableArray` может быть эффективнее (редко).

---

### NSMutableArray vs Swift Array

| Характеристика                        | `Array<Element>`                              | `NSMutableArray`                                         |
| ------------------------------------- | --------------------------------------------- | -------------------------------------------------------- |
| **Тип**                               | Структура ([[value type]])                    | Класс ([[reference type]])                               |
| **Мутабельность**                     | `var` для изменения, `let` для неизменяемости | Всегда мутабельный (отдельный `NSArray` — иммутабельный) |
| **Тип элементов**                     | [[Generic]], строгий (`<Element>`)            | [[AnyObject]] (объекты)                                  |
| **Value types (Int, String, struct)** | ✅ (хранятся напрямую)                         | ❌ (требуют обёртки в NSObject)                           |
| **Copy-on-Write**                     | ✅                                             | ❌ (копирование — это отдельный вызов)                    |
| **Многопоточность**                   | Не потокобезопасен (как и `NSMutableArray`)   | Не потокобезопасен                                       |
| **Совместимость с Objective-C**       | ❌ (требуется мост)                            | ✅ (родной)                                               |

---

### Создание NSMutableArray

#### 1. **Пустой массив**

```swift
import Foundation

let emptyArray = NSMutableArray()
let emptyArrayWithCapacity = NSMutableArray(capacity: 10)  // с начальной ёмкостью
```

#### 2. **С начальными объектами**

```swift
let array = NSMutableArray(objects: "Apple", "Banana", "Orange")
```

#### 3. **Из [[Swift]] Array**

```swift
let swiftArray = ["A", "B", "C"]
let nsMutableArray = NSMutableArray(array: swiftArray)
```

#### 4. **С литералом (только для NSArray, требуется приведение)**

```swift
// NSMutableArray не поддерживает литералы напрямую
let array: NSMutableArray = ["A", "B", "C"]  // Работает через мост
```

---

### Базовые операции

#### Добавление элементов

```swift
let array = NSMutableArray()

// Добавление одного элемента
array.add("Apple")

// Добавление нескольких элементов
array.addObjects(from: ["Banana", "Orange"])

// Вставка по индексу
array.insert("Mango", at: 1)

// Замена элемента
array.replaceObject(at: 0, with: "Apricot")
```

#### Удаление элементов

```swift
// Удаление по индексу
array.removeObject(at: 0)

// Удаление последнего
array.removeLastObject()

// Удаление конкретного объекта
array.remove("Banana")

// Удаление всех объектов
array.removeAllObjects()

// Удаление объектов в диапазоне
array.removeObjects(in: NSRange(location: 0, length: 2))
```

#### Доступ к элементам

```swift
// По индексу
let first = array.object(at: 0)
let firstShort = array[0]  // современный сахар

// Количество элементов
let count = array.count

// Индекс объекта
let index = array.index(of: "Orange")
```

---

### Итерация

```swift
// for-in
for item in array {
    print(item)
}

// with objects
array.enumerateObjects { object, index, stop in
    print("\(index): \(object)")
}

// with block
array.enumerateObjects(options: .concurrent) { object, index, stop in
    // параллельная итерация
}
```

---

### Преобразование в Swift Array

```swift
let nsArray: NSMutableArray = ["A", "B", "C"]

// Небезопасно (может быть nil, если типы не совпадают)
let swiftArray = nsArray as? [String] ?? []

// Безопасно (если знаете тип)
let safeArray = nsArray.compactMap { $0 as? String }
```

---

### NSMutableArray и Value Types

`NSMutableArray` может хранить только объекты. Для хранения value types ([[Int]], [[String]], [[struct]]) они автоматически оборачиваются в `NSObject`.

```swift
let array = NSMutableArray()

// Int -> NSNumber
array.add(42)

// String -> NSString
array.add("Hello")

// Bool -> NSNumber
array.add(true)

// Struct -> не работает напрямую
struct Point {
    let x: Int
    let y: Int
}
// array.add(Point(x: 1, y: 2))  // ❌ Ошибка

// Нужно обернуть в NSObject или использовать Swift Array
```

---

### NSMutableArray vs NSArray

| Характеристика | `NSArray` | `NSMutableArray` |
|---|---|---|
| **Мутабельность** | Иммутабельный | Мутабельный |
| **Производительность чтения** | Высокая | Высокая |
| **Производительность записи** | — | Ниже (из-за копирования при изменении) |
| **Потокобезопасность** | Да (только чтение) | Нет |
| **Копирование** | Дешёвое (поверхностное) | Требует `mutableCopy` |

---

### Копирование и мутабельность

```swift
let original = NSMutableArray(array: ["A", "B", "C"])

// Неглубокое копирование (тот же объект)
let same = original  // ссылается на тот же массив

// Копия (иммутабельная)
let copy = original.copy() as! NSArray
// copy.add("D")  // ❌ Ошибка: мутабельная операция на NSArray

// Мутабельная копия
let mutableCopy = original.mutableCopy() as! NSMutableArray
mutableCopy.add("D")  // ✅
```

---

### NSMutableArray в Objective-C (пример)

```objc
NSMutableArray *array = [NSMutableArray array];
[array addObject:@"Apple"];
[array addObject:@"Banana"];
[array removeObjectAtIndex:0];
NSLog(@"%@", array);  // ("Banana")
```

---

### Производительность

| Операция | Сложность |
|---|---|
| **Доступ по индексу** | O(1) |
| **Добавление в конец** | O(1) амортизированная |
| **Вставка в начало/середину** | O(n) |
| **Удаление по индексу** | O(n) |
| **Поиск элемента** | O(n) |
| **Копирование** | O(n) |

---

### Когда стоит использовать NSMutableArray в Swift

| Сценарий | Использовать | Почему |
|---|---|---|
| **Новый проект** | ❌ (используй `Array`) | Быстрее, безопаснее, типобезопаснее |
| **Legacy Objective-C код** | ✅ | Совместимость |
| **Core Data (NSOrderedSet)** | ✅ | Некоторые API возвращают `NSMutableArray` |
| **Objective-C runtime** | ✅ | Для низкоуровневых операций |
| **macOS AppKit** | ✅ | Некоторые старые API ожидают `NSMutableArray` |

---

### Лучшие практики

1.  **В Swift предпочитай нативный `Array`** — он типобезопаснее и производительнее.
2.  **При получении `NSMutableArray` из API сразу преобразуй в Swift Array.**
3.  **Не смешивай `NSMutableArray` и Swift Array без необходимости** — постоянное мостовое преобразование дорого.
4.  **Для многопоточного доступа используй синхронизацию** — `NSMutableArray` не потокобезопасен.
5.  **При передаче в Objective-C методы используй `[NSMutableArray arrayWithArray:]` или `mutableCopy`.**

```swift
// Плохо: постоянное преобразование туда-обратно
var array = [1, 2, 3]
let mutableArray = NSMutableArray(array: array)
mutableArray.add(4)
array = mutableArray as? [Int] ?? []

// Хорошо: остаёмся в Swift Array
var array = [1, 2, 3]
array.append(4)
```

---

### Итог

**`NSMutableArray`** — это класс Foundation для работы с мутабельными массивами объектов. В современной Swift-разработке его использование сводится к минимуму:

1.  **Для новых проектов всегда используй нативный `Array`.**
2.  **`NSMutableArray` нужен только для совместимости с Objective-C.**
3.  **При получении `NSMutableArray` из внешнего API сразу конвертируй в Swift Array.**
4.  **Помни о том, что `NSMutableArray` — reference type, а `Array` — value type.**

Понимание `NSMutableArray` важно для поддержки легаси-кода и глубокого понимания Foundation, но в повседневной Swift-разработке он почти не встречается.