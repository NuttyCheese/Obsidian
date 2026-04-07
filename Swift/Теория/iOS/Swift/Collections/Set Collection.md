#swift #set #collection #hashable #unordered #unique #data-structures

---
### Определение
**Set** — это неупорядоченная коллекция уникальных элементов в [[Swift]]. В отличие от массивов ([[Array]]), множество не допускает дубликатов и не гарантирует порядок хранения элементов. Set является **[[value type]]** (как и все коллекции в Swift) и использует хеш-таблицу для быстрого доступа к элементам.

Основное преимущество Set — **мгновенный поиск элемента** (O(1) в среднем), что делает его идеальным выбором для проверки наличия элемента, удаления дубликатов и выполнения операций над множествами (пересечение, объединение, разность).

### Зачем это знать iOS-разработчику?
1.  **Уникальность элементов:** Гарантирует, что каждый элемент присутствует только один раз.
2.  **Быстрый поиск:** Проверка `contains` выполняется за константное время O(1).
3.  **Теория множеств:** Поддержка пересечения, объединения, вычитания и симметрической разности.
4.  **Оптимизация:** Удаление дубликатов из массивов через преобразование в Set.
5.  **Сравнение коллекций:** Легко найти общие или уникальные элементы между двумя коллекциями.

---

### Основные характеристики

| Характеристика          | Set         | [[Array]]         | [[Dictionary]]    |
| ----------------------- | ----------- | ----------------- | ----------------- |
| **Упорядоченность**     | Нет         | Да                | Нет               |
| **Уникальность ключей** | Да          | Нет               | Да (ключи)        |
| **Доступ по индексу**   | Нет         | Да (O(1))         | Нет (по ключу)    |
| **Поиск элемента**      | O(1)        | O(n)              | O(1) (по ключу)   |
| **Хранение**            | Хеш-таблица | Непрерывный буфер | Хеш-таблица       |
| **Требования к типу**   | `Hashable`  | Нет               | `Hashable` (ключ) |

---

### Создание Set

#### 1. **Пустое множество**

```swift
var emptySet = Set<String>()
var anotherEmpty: Set<Int> = []
```

#### 2. **С литералом массива**

```swift
let fruits: Set = ["Apple", "Banana", "Orange"]
// тип выводится как Set<String>
```

#### 3. **Из массива (удаление дубликатов)**

```swift
let numbers = [1, 2, 2, 3, 3, 3, 4]
let uniqueNumbers = Set(numbers)
print(uniqueNumbers)  // [4, 2, 3, 1] (порядок не гарантирован)
```

#### 4. **С указанием типа**

```swift
let intSet: Set<Int> = [1, 2, 3]
let stringSet = Set<String>(["a", "b", "c"])
```

---

### Основные операции

#### Добавление и удаление

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

#### Проверка наличия

```swift
let numbers: Set = [1, 2, 3, 4, 5]

numbers.contains(3)   // true
numbers.contains(10)  // false
```

#### Количество элементов

```swift
let items: Set = ["A", "B", "C"]
print(items.count)     // 3
print(items.isEmpty)   // false
```

#### Итерация

```swift
let animals: Set = ["Cat", "Dog", "Bird"]

for animal in animals {
    print(animal)  // порядок не гарантирован
}

// Сортировка при итерации
for animal in animals.sorted() {
    print(animal)  // Bird, Cat, Dog
}
```

---

### Операции над множествами (Set Algebra)

#### 1. **Пересечение (intersection)**

```swift
let setA: Set = [1, 2, 3, 4]
let setB: Set = [3, 4, 5, 6]

let intersection = setA.intersection(setB)  // [3, 4]
```

#### 2. **Объединение (union)**

```swift
let union = setA.union(setB)  // [1, 2, 3, 4, 5, 6]
```

#### 3. **Разность (subtracting)**

```swift
let difference = setA.subtracting(setB)  // [1, 2]
```

#### 4. **Симметрическая разность (symmetricDifference)**

```swift
let symmetricDiff = setA.symmetricDifference(setB)  // [1, 2, 5, 6]
```

#### 5. **Модифицирующие версии**

```swift
var mutableSet: Set = [1, 2, 3, 4]

mutableSet.formIntersection([3, 4, 5])   // mutableSet = [3, 4]
mutableSet.formUnion([6, 7])             // mutableSet = [3, 4, 6, 7]
mutableSet.subtract([3, 6])              // mutableSet = [4, 7]
mutableSet.formSymmetricDifference([7, 8]) // mutableSet = [4, 8]
```

---

### Проверка отношений между множествами

#### 1. **Равенство**

```swift
let set1: Set = [1, 2, 3]
let set2: Set = [3, 2, 1]
let set3: Set = [1, 2, 4]

set1 == set2  // true (порядок не важен)
set1 == set3  // false
```

#### 2. **Подмножество (subset)**

```swift
let allNumbers: Set = [1, 2, 3, 4, 5]
let smallSet: Set = [1, 2]

smallSet.isSubset(of: allNumbers)     // true
allNumbers.isSuperset(of: smallSet)   // true
```

#### 3. **Строгое подмножество (strict subset)**

```swift
let all: Set = [1, 2, 3]
let same: Set = [1, 2, 3]
let smaller: Set = [1, 2]

smaller.isStrictSubset(of: all)   // true
same.isStrictSubset(of: all)      // false (равны)
```

#### 4. **Пересекающиеся множества (intersects)**

```swift
let setA: Set = [1, 2, 3]
let setB: Set = [3, 4, 5]
let setC: Set = [6, 7, 8]

setA.intersects(setB)   // true
setA.intersects(setC)   // false
```

#### 5. **Дизъюнктные множества (isDisjoint)**

```swift
setA.isDisjoint(with: setC)  // true (не пересекаются)
setA.isDisjoint(with: setB)  // false (пересекаются)
```

---

### Требование [[Hashable]]

Чтобы тип мог храниться в Set, он должен соответствовать протоколу `Hashable`. Все базовые типы ([[Int]], [[String]], [[Double]], [[Bool]]) уже реализуют `Hashable`.

#### Кастомный тип в Set

```swift
struct User: Hashable {
    let id: Int
    let name: String
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)  // только id для уникальности
    }
    
    static func == (lhs: User, rhs: User) -> Bool {
        return lhs.id == rhs.id
    }
}

let users: Set = [
    User(id: 1, name: "Alice"),
    User(id: 2, name: "Bob"),
    User(id: 1, name: "Alice")  // не добавится (id совпадает)
]
print(users.count)  // 2
```

---

### Set и производительность

#### Сравнение с Array

```swift
import Darwin

let array = Array(0..<10_000)
let set = Set(array)

func measure(_ name: String, _ block: () -> Void) {
    let start = mach_absolute_time()
    block()
    let end = mach_absolute_time()
    var info = mach_timebase_info()
    mach_timebase_info(&info)
    let elapsed = (end - start) * UInt64(info.numer) / UInt64(info.denom)
    print("\(name): \(elapsed / 1_000) мкс")
}

measure("Array contains") {
    _ = array.contains(9_999)  // O(n)
}

measure("Set contains") {
    _ = set.contains(9_999)    // O(1)
}
// Set содержит намного быстрее для больших коллекций
```

---

### Преобразование между Set и Array

```swift
// Array → Set (удаление дубликатов)
let arrayWithDuplicates = [1, 2, 2, 3, 3, 3]
let uniqueSet = Set(arrayWithDuplicates)  // [1, 2, 3]

// Set → Array
let set: Set = [1, 2, 3]
let array = Array(set)  // [2, 3, 1] (порядок не гарантирован)
let sortedArray = set.sorted()  // [1, 2, 3]
```

---

### Set и другие коллекции [[SwiftUI]]

```swift
import SwiftUI

struct FilterView: View {
    let allCategories: Set = ["Fruits", "Vegetables", "Dairy", "Meat"]
    @State private var selectedCategories: Set<String> = []
    
    var body: some View {
        List {
            ForEach(allCategories.sorted(), id: \.self) { category in
                HStack {
                    Text(category)
                    Spacer()
                    if selectedCategories.contains(category) {
                        Image(systemName: "checkmark")
                    }
                }
                .contentShape(Rectangle())
                .onTapGesture {
                    if selectedCategories.contains(category) {
                        selectedCategories.remove(category)
                    } else {
                        selectedCategories.insert(category)
                    }
                }
            }
        }
    }
}
```

---

### Полезные методы

#### `randomElement()`

```swift
let colors: Set = ["Red", "Green", "Blue"]
let randomColor = colors.randomElement()
```

#### [[sort|sorted()]]

```swift
let numbers: Set = [5, 2, 8, 1]
let sortedNumbers = numbers.sorted()  // [1, 2, 5, 8]
```

#### [[map]], [[filter]], [[reduce]]

```swift
let numbers: Set = [1, 2, 3, 4, 5]

let squares = numbers.map { $0 * $0 }           // [1, 4, 9, 16, 25]
let evens = numbers.filter { $0 % 2 == 0 }      // [2, 4]
let sum = numbers.reduce(0, +)                  // 15
```

---

### Лучшие практики

#### 1. **Используйте Set для проверки уникальности**

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

#### 2. **Используйте Set для быстрого поиска**

```swift
let validTokens: Set = ["abc123", "def456", "ghi789"]

func isValidToken(_ token: String) -> Bool {
    return validTokens.contains(token)  // O(1)
}
```

#### 3. **Используйте Set для операций над множествами**

```swift
let activeUsers: Set = [1, 2, 3, 4]
let premiumUsers: Set = [2, 4, 6]

let nonPremiumActive = activeUsers.subtracting(premiumUsers)  // [1, 3]
```

#### 4. **Не полагайтесь на порядок**

```swift
// ❌ Плохо — порядок не гарантирован
let items: Set = ["A", "B", "C"]
print(items.first)  // Может быть "A", "B" или "C"

// ✅ Хорошо — явная сортировка
for item in items.sorted() {
    print(item)
}
```

#### 5. **Для кастомных типов продумайте hash(into:)**

```swift
struct Product: Hashable {
    let id: Int
    let name: String
    let price: Decimal
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)  // Только id для идентификации
    }
}
```

---

### Короткое правило

> **Set** — неупорядоченная коллекция уникальных элементов.  
> Используйте для проверки наличия, удаления дубликатов и операций над множествами.  
> Требует `Hashable`. Поиск — O(1).

### Итог

**Set** в Swift:

1.  **Хранит уникальные элементы** — дубликаты автоматически исключаются.
2.  **Не гарантирует порядок** — элементы не индексированы.
3.  **Быстрый поиск** — `contains` работает за O(1).
4.  **Поддерживает операции над множествами** — объединение, пересечение, разность.
5.  **Требует `Hashable`** — тип должен быть хешируемым.
6.  **Value type** — копируется при присваивании.

Set — незаменимый инструмент для работы с уникальными значениями и выполнения операций теории множеств в Swift.

---
## 1. Set — это хеш-таблица (Hash Table)

Под капотом `Set` — это **хеш-таблица с открытой адресацией** (open addressing).

Он НЕ хранит элементы в виде:

❌ Массива  
❌ Связного списка  
❌ Двоичного дерева  

✅ **Массив «корзин» (buckets)**, где каждая корзина — это опциональный элемент.

---

## 2. Упрощенная схема памяти

```text
Set «из коробки»:

Storage (хеш-таблица)
┌───────┬───────┬───────┬───────┬───────┬───────┐
│       │       │       │       │       │       │
│  nil  │  "B"  │  nil  │  "A"  │  nil  │  "C"  │
│       │       │       │       │       │       │
└───────┴───────┴───────┴───────┴───────┴───────┘
  0       1       2       3       4       5
```

- Размер массива — степень двойки (8, 16, 32 …)
- Индекс корзины = `hashValue % capacity`
- Если корзина занята — ищется следующая свободная (линейный пробинг)

---

## 3. Пример: вставка элемента

```swift
var set = Set<String>()
set.insert("A")
```

### Шаги под капотом:

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

---

## 4. Поиск элемента O(1) — почему так быстро

```swift
set.contains("A")
```

- Берём хеш `"A"`
- Вычисляем индекс
- Проверяем bucket[index]
- ✅ Нашли за 1–2 операции

🔁 Если корзина занята другим элементом (коллизия), идём дальше, но **в среднем** всё равно O(1).

---

## 5. Требование Hashable — почему оно нужно

Хеш-таблица не может работать без быстрого вычисления индекса.

```swift
protocol Hashable {
    func hash(into hasher: inout Hasher)
}
```

Что делает `Set` при вставке:

```swift
var hasher = Hasher()
element.hash(into: &hasher)
let hashValue = hasher.finalize()
```

Без `Hashable` Set не сможет ни найти, ни сохранить элемент.

---

## 6. Упрощенная реализация Set (учебная)

```swift
struct MySet<Element: Hashable> {
    private var storage: [Element?]
    private var count = 0
    
    init(capacity: Int = 8) {
        storage = Array(repeating: nil, count: capacity)
    }
    
    private func index(for element: Element) -> Int {
        let hash = element.hashValue
        return abs(hash) % storage.count
    }
    
    mutating func insert(_ element: Element) {
        let idx = index(for: element)
        
        if storage[idx] == nil {
            storage[idx] = element
            count += 1
        }
    }
    
    func contains(_ element: Element) -> Bool {
        let idx = index(for: element)
        return storage[idx] == element
    }
}
```

---

## 7. Сравнение структур памяти

| Коллекция   | Лежащая в основе структура               |
| ----------- | ---------------------------------------- |
| `Array`     | Непрерывный буфер в памяти               |
| `Dictionary`| Хеш-таблица (ключ → значение)            |
| **`Set`**   | **Хеш-таблица без значений, только ключи** |

> Set — это **Dictionary, у которого нет values**, только keys.

---

## 8. Итог: что важно знать про Set под капотом

| Характеристика            | Реальность                                 |
| ------------------------- | ------------------------------------------ |
| Структура                 | Хеш-таблица (открытая адресация)           |
| Индексация                | `hashValue % capacity`                     |
| Поиск / вставка           | O(1) в среднем                             |
| Память                    | Массив + свободные корзины                 |
| Порядок элементов         | Не гарантирован, зависит от хешей и истории |
| Увеличение размера        | При загрузке > 75% (rehashing)             |
| Почему нужен `Hashable`   | Для вычисления индекса                     |

---

## 9. Коротко — как Set НЕ выглядит

❌ Не массив с проверкой дубликатов  
❌ Не бинарное дерево  
❌ Не связанный список

✅ Хеш-таблица, где в корзинах лежат элементы

---

## 10. Вывод (для собеседований и понимания)

> **Set в Swift — это хеш-таблица с открытой адресацией, где ключи и есть сами элементы.**

Именно поэтому:
- ✅ Поиск, вставка, удаление — **быстрые**
- ❌ **Нет порядка**
- ✅ **Требуется Hashable**
- ✅ **Удаление дубликатов — бесплатно**
