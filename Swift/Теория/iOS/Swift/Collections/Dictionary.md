#swift #dictionary #collection #hashable #key-value #data-structures

---
### Определение
**Dictionary** — это коллекция, которая хранит данные в виде неупорядоченных пар «ключ — значение». Каждый ключ в словаре уникален, а значение может быть любого типа. Словари обеспечивают быстрый доступ к значениям по ключу (в среднем за O(1)) и широко используются для хранения конфигураций, данных из JSON, кэшей и других ассоциативных структур.

### Зачем это знать iOS-разработчику?
1.  **Быстрый доступ по ключу:** O(1) в среднем — идеально для поиска, кэшей, словарей.
2.  **Уникальность ключей:** Автоматически гарантирует, что каждый ключ встречается только один раз.
3.  **Сериализация:** Основа для работы с [[JSON]], [[UserDefaults]], Property Lists.
4.  **[[Value type]]:** Безопасность и предсказуемость при копировании.
5.  **Гибкость:** Может хранить значения любых типов, включая вложенные словари и массивы.

---

### Основные характеристики

| Характеристика          | Dictionary        | [[Array]] | [[Set Collection\|Set]] |
| ----------------------- | ----------------- | --------- | ----------------------- |
| **Упорядоченность**     | Нет               | Да        | Нет                     |
| **Уникальность ключей** | Да                | Нет       | Да (элементы)           |
| **Доступ по индексу**   | Нет (по ключу)    | Да        | Нет                     |
| **Поиск элемента**      | O(1) (по ключу)   | O(n)      | O(1)                    |
| **Требования к типу**   | Key: [[Hashable]] | Нет       | Element: `Hashable`     |

---

### Создание Dictionary

#### 1. **Пустой словарь**

```swift
var emptyDict = [String: Int]()
var anotherEmpty: [Int: String] = [:]
```

#### 2. **С литералом**

```swift
let capitals: [String: String] = [
    "France": "Paris",
    "Japan": "Tokyo",
    "Germany": "Berlin"
]
```

#### 3. **Из последовательности**

```swift
let pairs = [("a", 1), ("b", 2), ("c", 3)]
let dict = Dictionary(uniqueKeysWithValues: pairs)
print(dict)  // ["a": 1, "b": 2, "c": 3]
```

#### 4. **С значениями по умолчанию**

```swift
let keys = ["a", "b", "c"]
let dict = Dictionary(uniqueKeysWithValues: zip(keys, repeatElement(0, count: keys.count)))
print(dict)  // ["a": 0, "b": 0, "c": 0]
```

---

### Основные операции

#### Доступ и изменение

```swift
var scores = ["Alice": 10, "Bob": 20]

// Чтение (возвращает Optional)
let aliceScore = scores["Alice"]  // Optional(10)
let unknown = scores["Eve"]       // nil

// Добавление / обновление
scores["Charlie"] = 30   // добавляет
scores["Alice"] = 15     // обновляет

// Удаление
scores["Bob"] = nil      // удаляет ключ
scores.removeValue(forKey: "Charlie")  // возвращает Optional(30)

// Проверка наличия ключа
if scores.keys.contains("Alice") {
    print("Alice exists")
}
```

#### Количество элементов

```swift
let dict = ["a": 1, "b": 2, "c": 3]
print(dict.count)     // 3
print(dict.isEmpty)   // false
```

#### Итерация

```swift
let capitals = ["France": "Paris", "Japan": "Tokyo", "Germany": "Berlin"]

// По парам (ключ, значение)
for (country, capital) in capitals {
    print("\(country): \(capital)")
}

// Только ключи
for country in capitals.keys {
    print(country)
}

// Только значения
for capital in capitals.values {
    print(capital)
}

// Сортировка
for (country, capital) in capitals.sorted(by: { $0.key < $1.key }) {
    print("\(country): \(capital)")
}
```

---

### Полезные методы

#### `merge`

```swift
var dict = ["a": 1, "b": 2]
dict.merge(["b": 3, "c": 4]) { (current, new) in new }
print(dict)  // ["a": 1, "b": 3, "c": 4]
```

#### `mapValues`

```swift
let scores = ["Alice": 10, "Bob": 20]
let doubled = scores.mapValues { $0 * 2 }
print(doubled)  // ["Alice": 20, "Bob": 40]
```

#### `filter`

```swift
let filtered = scores.filter { $0.value >= 15 }
print(filtered)  // ["Bob": 20]
```

#### `compactMapValues`

```swift
let strings = ["a": "1", "b": "abc", "c": "3"]
let ints = strings.compactMapValues { Int($0) }
print(ints)  // ["a": 1, "c": 3]
```

#### `grouping`

```swift
let words = ["apple", "banana", "apricot", "blueberry"]
let grouped = Dictionary(grouping: words, by: { $0.first! })
print(grouped)  // ["a": ["apple", "apricot"], "b": ["banana", "blueberry"]]
```

---

### Значения по умолчанию и подписки

#### `default` параметр

```swift
let dict = ["a": 1, "b": 2]
let value = dict["c", default: 0]
print(value)  // 0
```

#### `updateValue` метод

```swift
var dict = ["a": 1, "b": 2]
let old = dict.updateValue(3, forKey: "a")
print(old)  // Optional(1)
print(dict) // ["a": 3, "b": 2]
```

---

### Вложенные словари

```swift
var userData: [String: [String: Any]] = [
    "Alice": [
        "age": 25,
        "city": "New York"
    ],
    "Bob": [
        "age": 30,
        "city": "London"
    ]
]

userData["Alice"]?["age"] = 26
print(userData["Alice"]?["age"] ?? 0)  // 26
```

---

### Dictionary и JSON

```swift
import Foundation

let jsonString = """
{
    "name": "Alice",
    "age": 25,
    "city": "New York"
}
"""

// JSON → Dictionary
if let data = jsonString.data(using: .utf8),
   let dict = try? JSONSerialization.jsonObject(with: data) as? [String: Any] {
    print(dict["name"] ?? "")
}

// Dictionary → JSON
let dict: [String: Any] = ["name": "Alice", "age": 25]
if let data = try? JSONSerialization.data(withJSONObject: dict, options: .prettyPrinted),
   let json = String(data: data, encoding: .utf8) {
    print(json)
}
```

---

### Dictionary и UserDefaults

```swift
let defaults = UserDefaults.standard
let settings = ["theme": "dark", "notifications": true] as [String: Any]

defaults.set(settings, forKey: "userSettings")

if let loaded = defaults.dictionary(forKey: "userSettings") {
    print(loaded["theme"] as? String ?? "")
}
```

---

### Производительность

| Операция | Средняя сложность | Худшая сложность |
|----------|------------------|------------------|
| Доступ по ключу | O(1) | O(n) |
| Вставка | O(1) | O(n) |
| Удаление | O(1) | O(n) |
| Итерация | O(n) | O(n) |

**Примечание:** Худший случай (O(n)) возникает при сильных коллизиях хешей.

---

### Под капотом: устройство Dictionary

Swift Dictionary — это **хеш-таблица с открытой адресацией (open addressing)**. Она состоит из:

1.  **Бакеты (buckets)** — массив, где хранятся элементы.
2.  **Hash + Metadata** — для быстрого поиска и разрешения коллизий.

#### Структура бакета (упрощённо)

```text
struct Bucket {
    var hash: UInt       // часть hashValue для ускорения проверки
    var key: Key
    var value: Value
    var state: UInt8     // empty / occupied / tombstone
}
```

#### Принцип работы

1.  **Вставка:**
    - Вычисляется `hash(key)`
    - Индекс = `hash % capacity`
    - Если бакет пустой → вставляем
    - Если занят → **linear probing** (идём к следующему бакету)

2.  **Поиск:**
    - Вычисляется `hash(key)`
    - Линейный поиск до пустого бакета или нахождения ключа (сравнение через `==`)

3.  **Удаление:**
    - Находится элемент
    - Ставится **tombstone**, чтобы не нарушать цепочку поиска при коллизиях

#### Small Dictionary Optimization

- Для маленьких словарей (до 3–5 элементов) Swift хранит данные **inline** прямо внутри структуры Dictionary.
- Это позволяет избежать лишних heap-аллокаций и ускоряет копирование.

#### Rehashing (ресайз)

- Когда **load factor** превышает ~0.75:
    1.  Выделяется новый массив бакетов большего размера
    2.  Все элементы перехешируются
    3.  Старый массив освобождается

---

### Хеширование и Hashable

Ключи в Dictionary должны соответствовать протоколу `Hashable`. Встроенные типы ([[Int]], [[String]], [[Double]], [[Bool]]) уже реализуют его.

#### Кастомный тип как ключ

```swift
struct UserID: Hashable {
    let value: Int
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(value)
    }
    
    static func == (lhs: UserID, rhs: UserID) -> Bool {
        return lhs.value == rhs.value
    }
}

var users: [UserID: String] = [:]
users[UserID(value: 1)] = "Alice"
```

---

### Сравнение Dictionary и других коллекций

| Характеристика | Dictionary | Array | Set |
|----------------|------------|-------|-----|
| **Тип доступа** | По ключу | По индексу | По значению |
| **Упорядоченность** | Нет | Да | Нет |
| **Дубликаты** | Ключи уникальны | Да | Элементы уникальны |
| **Требование Hashable** | Ключи | Нет | Элементы |
| **Когда использовать** | Поиск по ключу | Последовательный доступ | Уникальность + поиск |

---

### Best Practices

#### 1. **Используйте default для безопасного доступа**

```swift
// ❌ Плохо
let value = dict[key] ?? 0

// ✅ Хорошо
let value = dict[key, default: 0]
```

#### 2. **Предпочитайте compactMapValues для преобразования**

```swift
let strings = ["a": "1", "b": "abc"]
let ints = strings.compactMapValues { Int($0) }  // ["a": 1]
```

#### 3. **Используйте Dictionary(grouping:) для группировки**

```swift
let grouped = Dictionary(grouping: items, by: { $0.category })
```

#### 4. **Для больших словарей задавайте начальную емкость**

```swift
var dict = [Int: String](minimumCapacity: 1000)
```

#### 5. **Не полагайтесь на порядок**

```swift
// ❌ Плохо — порядок не гарантирован
let firstKey = dict.keys.first

// ✅ Хорошо — явная сортировка
let sortedKeys = dict.keys.sorted()
```

---

### Короткое правило

> **Dictionary** — коллекция для хранения пар «ключ — значение» с быстрым доступом по ключу.  
> Используйте для конфигураций, кэшей, JSON-данных, поисковых таблиц.  
> Ключи должны быть `Hashable`, порядок не гарантирован, поиск O(1).

### Итог

**Dictionary** в Swift:

1.  **Value type** — копируется при присваивании.
2.  **Ключи уникальны** — дубликаты невозможны.
3.  **Доступ по ключу возвращает Optional** — ключ может отсутствовать.
4.  **Поддерживает** `mapValues`, `compactMapValues`, `merge`, `filter`, `grouping`.
5.  **Основа для JSON и UserDefaults.**
6.  **Под капотом — хеш-таблица с открытой адресацией** (open addressing).
7.  **Маленькие словари оптимизированы** (inline storage).

Понимание устройства и особенностей Dictionary помогает эффективно использовать его в проектах и избегать подводных камней с производительностью и памятью.