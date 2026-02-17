## 1. Введение

**Open Addressing** — это метод разрешения коллизий в хеш-таблицах.

Напомню, коллизия — это ситуация, когда два разных ключа попадают в одну и ту же ячейку (индекс) хеш-таблицы.

Open addressing отличается от **связанных списков ([[Chaining]])** тем, что:

- Все элементы **хранятся прямо в массиве бакетов**,
    
- Если ячейка занята, ищем **следующую свободную ячейку** по определённому правилу,
    
- **Списков нет** — только один массив.
    

Именно этот подход использует [[Swift]] для [[Dictionary]] и [[Set]].

---

## 2. Основная идея

Хеш-таблица с open addressing хранит элементы так:

```
table = [bucket0, bucket1, bucket2, bucket3, bucket4]
```

Каждый bucket может быть:

- пустым ([[nil]] / empty)
    
- занятым (`occupied`)
    
- помеченным как удалённый ([[tombstone]])
    

### Пример:

```text
table: [nil, nil, nil, nil, nil]
insert "apple" → hash = 1 → table[1] = "apple"
insert "banana" → hash = 1 → table[1] занят → ищем следующий свободный → table[2] = "banana"
```

---

## 3. Методы поиска свободной ячейки (probes)

### 3.1 Linear probing (линейный пробинг)

Идём по массиву по порядку:

```text
index, index+1, index+2, ...
```

Плюсы:

- очень просто
    
- легко реализовать
    

Минусы:

- формируются **кластеры**
    
- lookup может замедлиться
    

---

### 3.2 Quadratic probing (квадратичный пробинг)

```text
index + 1², index + 2², index + 3², ...
```

- уменьшает «кластеры»
    
- но сложнее вычислять
    

---

### 3.3 Double hashing

```text
index + hash2(key)
```

- каждая коллизия использует **другую хеш-функцию**
    
- минимизирует кластеры
    
- чуть дороже в вычислениях
    

---

## 4. Tombstones (надгробия)

При удалении элемента в open addressing **нельзя просто очистить ячейку**, иначе поиск других элементов, которые коллизировали в этом районе, может сломаться.

Решение:

- ставим специальный маркер: **tombstone**
    
- при вставке новый элемент может занять эту ячейку
    
- при поиске tombstone считается «занято», чтобы не остановить поиск
    

---

## 5. Пример на Swift (упрощённая реализация)

```swift
struct OpenAddressingHashTable<Key: Hashable, Value> {
    private var buckets: [(Key, Value)?]
    private(set) var count = 0
    
    init(capacity: Int) {
        buckets = Array(repeating: nil, count: capacity)
    }
    
    private func index(for key: Key) -> Int {
        return abs(key.hashValue) % buckets.count
    }
    
    mutating func insert(_ key: Key, value: Value) {
        var idx = index(for: key)
        
        while let bucket = buckets[idx] {
            if bucket.0 == key {
                buckets[idx] = (key, value)
                return
            }
            idx = (idx + 1) % buckets.count // linear probing
        }
        buckets[idx] = (key, value)
        count += 1
    }
    
    func value(for key: Key) -> Value? {
        var idx = index(for: key)
        
        while let bucket = buckets[idx] {
            if bucket.0 == key {
                return bucket.1
            }
            idx = (idx + 1) % buckets.count
        }
        return nil
    }
}
```

Пример использования:

```swift
var table = OpenAddressingHashTable<String, Int>(capacity: 5)
table.insert("apple", value: 1)
table.insert("banana", value: 2)
table.insert("cherry", value: 3)

print(table.value(for: "banana")) // 2
```

---

## 6. Преимущества open addressing

1. **Отличная cache locality**
    
    - Все элементы находятся в одном массиве
        
    - Быстрее, чем связанные списки ([[ARC]] + [[Heap]])
        
2. **Нет дополнительных аллокаций**
    
    - [[Linked List]] требует выделения узлов в heap
        
3. **Высокая производительность**
    
    - Lookup, insert и delete обычно O(1)
        

---

## 7. Минусы open addressing

1. **Сложнее реализация**
    
2. **Падение производительности при высокой загрузке**
    
    - Load factor > 0.7 → больше коллизий → больше probing
        
3. **Удаление сложнее**
    
    - Требуются tombstones
        
4. **Требует качественной хеш-функции**
    

---

## 8. Как Swift Dictionary использует open addressing

- Swift `Dictionary` хранит элементы **непосредственно в массиве бакетов**
    
- Коллизии разрешаются через **linear probing** + tombstones
    
- При достижении load factor → **ресайз и rehashing**
    
- Для стандартных типов ([[Int]], [[String]]) используется **встроенный качественный хеш**
    

---

## 9. Коллизии и производительность

При качественном hash:

- lookup: O(1) среднее
    
- insert: O(1) среднее
    
- delete: O(1) среднее
    

При плохом hash или высокой load factor:

- все операции могут деградировать до O(n)
    
- Swift [[Dictionary]] автоматически ресайзит массив и перераспределяет элементы
    

---

## 10. Советы для Swift-разработчика

1. **Использовать качественные [[Hashable]] типы**  
    Не делайте:
    
    ```swift
    func hash(into hasher: inout Hasher) {
        hasher.combine(0)
    }
    ```
    
2. **Не бояться коллизий**
    
    - Swift гарантирует корректное поведение через `==`
        
3. **Понимать, что ARC может влиять**
    
    - Все элементы [[Reference Type]] → [[retain]]/[[release]]
        
4. **Не пытаться реализовывать chaining в Swift**
    
    - Open addressing быстрее и оптимизирована
        

---

## 11. Короткий итог

- Open addressing — метод разрешения коллизий **без списков**
    
- Все элементы хранятся в массиве
    
- При коллизии ищем следующую свободную ячейку (probing)
    
- Tombstones решают проблему удаления
    
- Swift [[Dictionary]]/[[Set]] используют именно его
    
- Хороший hash → почти всегда O(1)
    
- Плохой hash → коллизии → падение производительности
    

---
