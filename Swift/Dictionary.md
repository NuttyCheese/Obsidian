**`Dictionary`** — это **коллекция, которая хранит данные в виде пар «ключ → значение»**.

- Каждый **ключ уникален** в словаре
    
- Значения можно извлекать через ключ
    
- Можно изменять, добавлять и удалять элементы
    
- Swift словарь является **[[Value Type]]** (копируется при присваивании)
    

> Проще говоря: `Dictionary` = «карта, где каждому ключу соответствует значение».

---

## 2. Основные термины

| Термин                  | Описание                                                                               |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **Key**                 | Тип значения, который используется для идентификации элемента (должен быть `Hashable`) |
| **Value**               | Тип значения, связанного с ключом                                                      |
| **[[Hashable]]**        | Протокол, который позволяет объекту быть ключом словаря                                |
| **Subscript**           | Доступ к элементам словаря через `dict[key]`                                           |
| **Mutable / Immutable** | `var` можно менять, `let` нельзя                                                       |
| **Optional Value**      | При извлечении через ключ возвращается `Value?`                                        |

---

## 3. Основной синтаксис

```swift
var ages: [String: Int] = ["Alice": 25, "Bob": 30]
let capitals: [String: String] = ["France": "Paris", "Japan": "Tokyo"]
```

- `[Key: Value]` → словарь с ключами и значениями
    
- [[Swift]] может **выводить тип автоматически** `var dict = ["x": 1, "y": 2]`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Создание и доступ к элементам

```swift
var fruits = ["Apple": 2, "Banana": 3]
print(fruits["Apple"]!) // 2

fruits["Orange"] = 5 // добавляем новый элемент
print(fruits) // ["Apple": 2, "Banana": 3, "Orange": 5]
```

- Доступ через `dict[key]` → возвращает `Optional`
    

---

### Пример 2. Обновление и удаление

```swift
var scores = ["Alice": 10, "Bob": 15]
scores["Alice"] = 20 // обновление
scores["Bob"] = nil   // удаление элемента
print(scores) // ["Alice": 20]
```

- Присвоение [[nil]] удаляет ключ
    

---

### Пример 3. Итерация по словарю

```swift
let capitals = ["France": "Paris", "Japan": "Tokyo"]

for (country, city) in capitals {
    print("\(country): \(city)")
}

// Output:
// France: Paris
// Japan: Tokyo
```

- Можно использовать [[for-in]] с **кортежами (ключ, значение)**
    

---

### Пример 4. Keys и Values

```swift
let ages = ["Alice": 25, "Bob": 30]

let names = Array(ages.keys)   // ["Alice", "Bob"]
let numbers = Array(ages.values) // [25, 30]

print(names)
print(numbers)
```

- `keys` и `values` возвращают **коллекции**, которые можно преобразовать в массив
    

---

### Пример 5. Dictionary с [[any]] / [[AnyObject]]

```swift
var mixed: [String: Any] = ["age": 25, "name": "Alice", "isStudent": true]
mixed["score"] = 95

var objects: [String: AnyObject] = ["person": Person(name: "Bob")]
```

- `[String: Any]` → можно хранить значения любых типов
    
- `[String: AnyObject]` → только объекты классов
    

---

### Пример 6. [[map]], [[filter]], [[reduce]] для словаря

```swift
let scores = ["Alice": 10, "Bob": 15, "Eve": 20]

let doubled = scores.mapValues { $0 * 2 }
let highScores = scores.filter { $0.value >= 15 }

print(doubled)    // ["Alice": 20, "Bob": 30, "Eve": 40]
print(highScores) // ["Bob": 15, "Eve": 20]
```

- `mapValues` → преобразует значения
    
- `filter` → фильтрует пары ключ-значение
    

---

## 5. Особенности Dictionary

1. **Value type** → копируется при присваивании
    
2. Ключи должны быть **Hashable**
    
3. Доступ по ключу возвращает **Optional**, потому что ключ может отсутствовать
    
4. Можно изменять, добавлять, удалять элементы
    
5. Поддерживает **итерацию, map, filter, reduce, keys, values**
    

---

## 6. Итог

- **Dictionary** = коллекция пар «ключ → значение»
    
- Используется для хранения и быстрого доступа к данным по ключу
    
- Подходит для: словарей, конфигураций, [[JSON]]-подобных структур, моделей данных
    

---

# под капотом

## 1. Основная идея

В Swift **`Dictionary`** — это не просто массив кортежей `(Key, Value)`.  
На самом деле это **хеш-таблица** с **[[open addressing]]** (открытая адресация) и оптимизациями под современный CPU.

Цель:

- Быстрый доступ по ключу → `O(1)` в среднем
    
- Разрешение коллизий → open addressing + tombstones
    
- Минимизация лишних аллокаций → встроенные буферы и inline storage
    

---

## 2. Как хранится Dictionary в памяти

В Swift `Dictionary` состоит из двух основных компонентов:

1. **Бакеты (Buckets)** — массив, где хранятся элементы
    
2. **Hash + Metadata** — данные для быстрого поиска и разрешения коллизий
    

### 2.1 Bucket

Каждый **bucket** содержит:

- **hash** ключа (часть hashValue, чтобы ускорить проверку)
    
- **ключ** (Key)
    
- **значение** (Value)
    
- Флаги состояния (empty / occupied / tombstone)
    

### Пример структуры бакета (упрощённо):

```text
struct Bucket {
    var hash: UInt
    var key: Key
    var value: Value
    var state: UInt8 // empty / occupied / tombstone
}
```

---

### 2.2 Inline storage и small buffer optimization

- Для маленьких словарей Swift может хранить элементы **inline в структуре Dictionary**, чтобы **не выделять отдельный heap**.
    
- Для больших словарей элементы хранятся **в heap**, а структура `Dictionary` остаётся value-type (копирование только при изменении).
    

**Пример:**

```swift
var smallDict = ["a": 1, "b": 2]
// элементы могут быть прямо внутри структуры
```

- При расширении словаря → выделяется **heap**, элементы перехешируются.
    

---

## 3. Как работает вставка и поиск

1. **Вставка:**
    

- Вычисляем `hash(key)`
    
- Находим индекс: `hash % capacity`
    
- Если бакет пустой → вставляем
    
- Если занят → **open addressing** (linear probing) ищем следующий свободный бакет
    

2. **Поиск:**
    

- Вычисляем `hash(key)`
    
- Идём по бакетам (probes) пока не найдём ключ или пустой бакет
    

3. **Удаление:**
    

- Ставим **tombstone**, чтобы не ломать поиск коллизий
    

---

## 4. Разрешение коллизий

Swift Dictionary использует:

- **Open Addressing**
    
- **Linear probing** + tombstones
    
- Проверка `==` ключей для окончательного сравнения
    

Пример коллизии:

```swift
struct BadHash: Hashable {
    let value: Int
    func hash(into hasher: inout Hasher) {
        hasher.combine(0)
    }
}

var dict = [BadHash: String]()
dict[BadHash(value: 1)] = "One"
dict[BadHash(value: 2)] = "Two"
// оба ключа попали в один бакет → линейный probing
```

- Lookup корректно находит каждый ключ через `==`
    

---

## 5. Размер в памяти (байтовый вес)

Размер словаря зависит от:

- Кол-ва элементов
    
- Размеров Key и Value
    
- Внутренней capacity (обычно больше, чем count для минимизации коллизий)
    

### Упрощённый расчёт

```text
Memory = sizeof(Bucket) * capacity
sizeof(Bucket) ≈ sizeof(Key) + sizeof(Value) + sizeof(UInt) + 1 байт флага
```

Пример:

```swift
let dict = ["a": 1, "b": 2] // Key = String, Value = Int
```

- `String` (маленький, inline) ~ 16 байт
    
- `Int` = 8 байт
    
- Hash = 8 байт
    
- Flag = 1 байт
    

**Итого один бакет ~ 33 байта** → округляется до 40 байт из-за выравнивания CPU.

**Для 2 элементов** → массив бакетов может иметь capacity = 4 → 160 байт.

---

### Примечание:

- Swift **не выделяет каждый Key/Value отдельно** в heap, если это возможно (inline storage)
    
- Для больших объектов Key/Value → heap аллокации, ARC retain/release
    

---

## 6. Small Dictionary Optimization

- Для маленьких словарей Swift хранит **до 3–5 элементов прямо внутри структуры**
    
- Позволяет:
    
    - Не выделять heap
        
    - Быстро копировать словарь (value-type semantics)
        

Пример:

```swift
var small = ["x": 1, "y": 2]
// inline buffer внутри Dictionary struct
```

- При добавлении элементов → **heap аллокация** с capacity ≥ count * 2
    

---

## 7. Rehashing (ресайз таблицы)

Когда **load factor** превышает ~0.7–0.75:

1. Выделяется новый массив бакетов большего размера
    
2. Все элементы перехешируются
    
3. Старый массив освобождается
    

- Это дорогая операция → поэтому лучше заранее подбирать capacity
    

---

## 8. Схема работы Dictionary

```
Key → hash(key) → index = hash % capacity → bucket[index]
  │
  └─> если коллизия → probing → следующий бакет
  │
  └─> сравнение key через == → Value найден
```

---

## 9. Итог

- Swift Dictionary = **Value type + hash table с open addressing**
    
- [[Коллизии]] → **разрешаются через linear probing**
    
- Memory-efficient: **inline storage + heap для больших таблиц**
    
- Memory footprint зависит от **capacity, Key/Value size и load factor**
    
- Производительность Lookup/Insert/Delete ~ O(1) среднее, O(n) в худшем случае
    

> Понимание внутренностей словаря полезно для: оптимизации, оценки памяти и производительности при работе с большими словарями или кастомными типами Key/Value.

---
