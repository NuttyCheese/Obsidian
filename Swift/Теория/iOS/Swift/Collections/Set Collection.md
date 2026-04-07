#swift #set #collection #hashable #unordered #unique #data-structures

---
## 1. Что такое Set

**Set** — это **неупорядоченная коллекция уникальных элементов**.

- Каждый элемент может встречаться **только один раз**
- Доступ к элементам — **не по индексу, а по самому значению**
- Элементы не имеют порядка
- Swift Set является **[[Value Type]]** (копируется при присваивании)

> Проще говоря: `Set` = «мешок уникальных значений, порядок которых не важен».

---

## 2. Основные термины

| Термин                   | Описание                                                       |
| ------------------------ | -------------------------------------------------------------- |
| **Element**              | Тип значения, хранящегося в множестве (должен быть `Hashable`) |
| **Hashable**             | Протокол, позволяющий быстро искать и сравнивать элементы      |
| **Member**               | Элемент множества                                              |
| **Intersection**         | Пересечение двух множеств                                      |
| **Union**                | Объединение двух множеств                                      |
| **Subtracting**          | Разность множеств                                              |
| **Symmetric Difference** | Элементы, входящие только в одно из множеств                   |

---

## 3. Создание Set

```swift
// Пустое множество
var emptySet = Set<String>()
var anotherEmpty: Set<Int> = []

// С литералом
let fruits: Set = ["Apple", "Banana", "Orange"]

// Из массива (удаляет дубликаты)
let numbers = [1, 2, 2, 3, 3, 3]
let uniqueNumbers = Set(numbers) // [1, 2, 3]

// С указанием типа
let intSet: Set<Int> = [1, 2, 3]
```

---

## 4. Основные операции

### 4.1 Добавление и удаление

```swift
var colors: Set = ["Red", "Green", "Blue"]

// Добавление
colors.insert("Yellow")      // (inserted: true, memberAfterInsert: "Yellow")
colors.insert("Red")         // (inserted: false, memberAfterInsert: "Red")

// Удаление
colors.remove("Green")       // Optional("Green")
colors.remove("Purple")      // nil
colors.removeAll()
```

### 4.2 Проверка наличия

```swift
let numbers: Set = [1, 2, 3, 4, 5]

numbers.contains(3)   // true
numbers.contains(10)  // false
```

### 4.3 Количество элементов

```swift
let items: Set = ["A", "B", "C"]
items.count    // 3
items.isEmpty  // false
```

### 4.4 Итерация

```swift
let animals: Set = ["Cat", "Dog", "Bird"]

for animal in animals {
    print(animal) // порядок не гарантирован
}

// С сортировкой
for animal in animals.sorted() {
    print(animal) // Bird, Cat, Dog
}
```

---

## 5. Операции над множествами

### 5.1 Пересечение (intersection)

```swift
let setA: Set = [1, 2, 3, 4]
let setB: Set = [3, 4, 5, 6]

let intersection = setA.intersection(setB) // [3, 4]
```

### 5.2 Объединение (union)

```swift
let union = setA.union(setB) // [1, 2, 3, 4, 5, 6]
```

### 5.3 Разность (subtracting)

```swift
let difference = setA.subtracting(setB) // [1, 2]
```

### 5.4 Симметрическая разность (symmetricDifference)

```swift
let symmetric = setA.symmetricDifference(setB) // [1, 2, 5, 6]
```

### 5.5 Модифицирующие версии

```swift
var mutableSet: Set = [1, 2, 3, 4]

mutableSet.formIntersection([3, 4, 5])   // mutableSet = [3, 4]
mutableSet.formUnion([6, 7])             // mutableSet = [3, 4, 6, 7]
mutableSet.subtract([3, 6])              // mutableSet = [4, 7]
mutableSet.formSymmetricDifference([7, 8]) // mutableSet = [4, 8]
```

---

## 6. Проверка отношений между множествами

```swift
let all: Set = [1, 2, 3, 4, 5]
let subset: Set = [1, 2]
let other: Set = [6, 7]

subset.isSubset(of: all)        // true
all.isSuperset(of: subset)      // true
subset.isStrictSubset(of: all)  // true

all.isDisjoint(with: other)     // true (не пересекаются)
all.intersects(subset)          // true (пересекаются)

all == [1, 2, 3, 4, 5]          // true
```

---

## 7. Преобразование в [[Array]]

```swift
let set: Set = [1, 2, 3]

// В массив
let array = Array(set)          // [2, 3, 1] (порядок не гарантирован)
let sortedArray = set.sorted()  // [1, 2, 3]

// Массив → Set (удаление дубликатов)
let withDups = [1, 2, 2, 3]
let unique = Set(withDups)      // [1, 2, 3]
```

---

## 8. Set и другие коллекции [[SwiftUI]]

```swift
import SwiftUI

struct FilterView: View {
    let allCategories: Set = ["Fruits", "Vegetables", "Dairy"]
    @State private var selected: Set<String> = []
    
    var body: some View {
        List(allCategories.sorted(), id: \.self) { category in
            HStack {
                Text(category)
                Spacer()
                if selected.contains(category) {
                    Image(systemName: "checkmark")
                }
            }
            .onTapGesture {
                if selected.contains(category) {
                    selected.remove(category)
                } else {
                    selected.insert(category)
                }
            }
        }
    }
}
```

---

## 9. Hashable и кастомные типы

Чтобы тип можно было хранить в Set, он должен быть [[Hashable]].

```swift
struct User: Hashable {
    let id: Int
    let name: String
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)  // только id определяет уникальность
    }
    
    static func == (lhs: User, rhs: User) -> Bool {
        return lhs.id == rhs.id
    }
}

let users: Set = [
    User(id: 1, name: "Alice"),
    User(id: 2, name: "Bob"),
    User(id: 1, name: "Alice") // не добавится
]
users.count // 2
```

---

## 10. Полезные методы

```swift
let set: Set = [1, 2, 3, 4, 5]

// randomElement
let random = set.randomElement()

// sorted
let sorted = set.sorted()

// map, filter, reduce
let squares = set.map { $0 * $0 }          // [1, 4, 9, 16, 25]
let evens = set.filter { $0 % 2 == 0 }     // [2, 4]
let sum = set.reduce(0, +)                 // 15
```

---

## 11. Под капотом: как устроен Set в Swift

**Set — это хеш-таблица без значений, только ключи.**

### 11.1 Структура памяти

Set — это **хеш-таблица с открытой адресацией (open addressing)**.

Он НЕ хранит элементы:
- ❌ в массиве
- ❌ в связном списке
- ❌ в бинарном дереве

✅ В массиве «бакетов» (buckets), где каждая корзина — это опциональный элемент.

```text
Set «из коробки»:

Storage (хеш-таблица)
┌───────┬───────┬───────┬───────┬───────┐
│  nil  │  "B"  │  nil  │  "A"  │  "C"  │
└───────┴───────┴───────┴───────┴───────┘
   0       1       2       3       4
```

### 11.2 Вставка элемента

```swift
var set = Set<String>()
set.insert("A")
```

Шаги:

1. Вычисляется хеш `"A"`:
   ```swift
   let hash = "A".hashValue // например, 123456789
   ```

2. Вычисляется индекс в таблице:
   ```swift
   let index = hash & (capacity - 1)   // capacity = 8
   // 123456789 & 7 → 5
   ```

3. Если bucket[5] == nil → кладём туда `"A"`

4. При загрузке > 75% → массив увеличивается (rehashing)

### 11.3 Поиск элемента

```swift
set.contains("A")
```

- Берётся хеш `"A"`
- Вычисляется индекс
- Проверяется bucket[index]
- Найдено за 1–2 операции

При коллизии (два элемента в одну корзину) — идём дальше (linear probing), но в среднем всё равно O(1).

### 11.4 Структура бакета (упрощённо)

```text
struct Bucket {
    var hash: UInt       // часть hashValue для ускорения проверки
    var element: Element
    var state: UInt8     // empty / occupied / tombstone
}
```

### 11.5 Small Set Optimization

- Для маленьких множеств (до 3–5 элементов) Swift может хранить элементы **inline** прямо внутри структуры Set
- Позволяет не выделять heap и быстро копировать

### 11.6 Почему нужен Hashable

Без `Hashable` Set не сможет вычислить индекс для элемента.

```swift
protocol Hashable {
    func hash(into hasher: inout Hasher)
}
```

### 11.7 Set vs Dictionary

| Характеристика     | Set                           | [[Dictionary]]       |
| ------------------ | ----------------------------- | -------------------- |
| Хранит             | только ключи                  | ключ → значение      |
| Уникальность       | элементов                     | ключей               |
| Когда использовать | множество уникальных значений | ассоциативный массив |

> Set — это **Dictionary, у которого нет значений, а есть только ключи**.

---

## 12. Производительность

| Операция | Средняя сложность | Худшая сложность |
|----------|------------------|------------------|
| `insert` | O(1) | O(n) |
| `remove` | O(1) | O(n) |
| `contains` | O(1) | O(n) |
| `union`/`intersection`/`subtracting` | O(m) | O(m × log n) |

**Худший случай (O(n))** возникает при сильных коллизиях хешей.

---

## 13. Best Practices

### 13.1 Используйте Set для проверки уникальности

```swift
// ✅ Хорошо
let uniqueIDs = Set(users.map { $0.id })

// ❌ Плохо
var uniqueIDs: [Int] = []
for user in users {
    if !uniqueIDs.contains(user.id) {
        uniqueIDs.append(user.id)
    }
}
```

### 13.2 Используйте Set для быстрого поиска

```swift
let validTokens: Set = ["abc123", "def456", "ghi789"]

func isValidToken(_ token: String) -> Bool {
    return validTokens.contains(token) // O(1)
}
```

### 13.3 Не полагайтесь на порядок

```swift
// ❌ Плохо
let items: Set = ["A", "B", "C"]
let first = items.first // непредсказуемо

// ✅ Хорошо
let sortedItems = items.sorted()
```

### 13.4 Для кастомных типов продумайте hash(into:)

```swift
struct Product: Hashable {
    let id: Int
    let name: String
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)  // только id определяет уникальность
    }
}
```

---

## 14. Короткое правило

> **Set** — неупорядоченная коллекция уникальных элементов.  
> Используйте для проверки наличия, удаления дубликатов и операций над множествами.  
> Требует `Hashable`. Поиск, вставка, удаление — O(1) в среднем.

---

## 15. Итог

**Set** в Swift:

1. **Value type** — копируется при присваивании
2. **Элементы уникальны** — дубликаты невозможны
3. **Неупорядочен** — не хранит индекс элементов
4. **Требует `Hashable`** — для быстрого поиска
5. **Поддерживает операции над множествами** — объединение, пересечение, разность
6. **Под капотом — хеш-таблица без значений**
7. **Маленькие множества оптимизированы** (inline storage)

Понимание устройства Set помогает эффективно использовать его для задач, связанных с уникальностью и быстрым поиском.