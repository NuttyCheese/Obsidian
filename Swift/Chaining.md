## 1. Введение

**Связанные списки** — это одна из классических стратегий **разрешения коллизий в хеш-таблицах**.

Когда два разных ключа попадают в одну ячейку хеш-таблицы:

```
hash(key1) % tableSize == hash(key2) % tableSize
```

- В **chaining** эта ячейка превращается в **список элементов**, все они сохраняются там.
    

> В отличие от **open addressing**, элементы не перемещаются по массиву, а «навешиваются» на один бакет.

---

## 2. Основная идея

Идея простая:

- Каждая ячейка хеш-таблицы содержит **ссылку на связанный список** (или [[nil]], если пусто)
    
- Если возникает коллизия, элемент просто **добавляется в список**
    
- Поиск идёт по списку внутри бакета
    

### Визуально

```
table[3] → ("apple", 10) → ("banana", 20) → nil
table[4] → ("orange", 5) → nil
table[5] → nil
```

- Индекс 3 содержит **цепочку из двух элементов**, потому что `apple` и `banana` коллизировали.
    

---

## 3. Реализация в [[Swift]] (упрощённо)

Сначала создадим узел списка:

```swift
class Node<Key: Hashable, Value> {
    let key: Key
    var value: Value
    var next: Node?

    init(key: Key, value: Value, next: Node? = nil) {
        self.key = key
        self.value = value
        self.next = next
    }
}
```

Теперь хеш-таблица с chaining:

```swift
class ChainingHashTable<Key: Hashable, Value> {
    private var buckets: [Node<Key, Value>?]
    private(set) var count = 0

    init(capacity: Int) {
        buckets = Array(repeating: nil, count: capacity)
    }

    private func index(for key: Key) -> Int {
        abs(key.hashValue) % buckets.count
    }

    func insert(_ key: Key, value: Value) {
        let idx = index(for: key)
        var node = buckets[idx]

        // обновляем значение, если ключ уже есть
        while let n = node {
            if n.key == key {
                n.value = value
                return
            }
            node = n.next
        }

        // вставляем новый узел в голову списка
        let newNode = Node(key: key, value: value, next: buckets[idx])
        buckets[idx] = newNode
        count += 1
    }

    func value(for key: Key) -> Value? {
        let idx = index(for: key)
        var node = buckets[idx]

        while let n = node {
            if n.key == key {
                return n.value
            }
            node = n.next
        }
        return nil
    }

    func remove(_ key: Key) {
        let idx = index(for: key)
        var node = buckets[idx]
        var prev: Node<Key, Value>? = nil

        while let n = node {
            if n.key == key {
                if let p = prev {
                    p.next = n.next
                } else {
                    buckets[idx] = n.next
                }
                count -= 1
                return
            }
            prev = node
            node = n.next
        }
    }
}
```

---

## 4. Пример использования

```swift
let table = ChainingHashTable<String, Int>(capacity: 5)
table.insert("apple", value: 10)
table.insert("banana", value: 20)
table.insert("orange", value: 30)

print(table.value(for: "banana")) // 20
table.remove("banana")
print(table.value(for: "banana")) // nil
```

- Если несколько ключей попали в один индекс, **они будут в цепочке**
    
- Lookup → перебор [[Linked List]] в бакете
    
- Insert → добавление в голову списка (или tail, по желанию)
    

---

## 5. Плюсы связанных списков

1. **Простая реализация**
    
    - Никаких хитрых probing-алгоритмов
        
2. **Не нужно пересаживать элементы**
    
    - Элемент остаётся в своём бакете
        
3. **Поддержка динамического размера**
    
    - Бакеты можно расширять, не трогая уже вставленные элементы
        
4. **Высокая устойчивость к плохому хешу**
    
    - Даже если много коллизий, элементы корректно хранятся в списке
        

---

## 6. Минусы связанных списков

1. **Снижение производительности при коллизиях**
    
    - Lookup становится O(n) внутри бакета
        
2. **Дополнительные аллокации в heap**
    
    - Каждый Node = отдельный объект → [[ARC]] [[retain]]/[[release]]
        
3. **Хуже локальность кэша**
    
    - Узлы могут быть разбросаны по памяти
        
4. **Сложность при параллельных вставках**
    
    - Нужна синхронизация при многопоточном доступе
        

---

## 7. Когда chaining используется

- В классических хеш-таблицах (например, учебники, Java HashMap до JDK 8)
    
- В ситуациях, где **простота важнее cache performance**
    
- Когда **много элементов и плохой hash** — chaining более стабильный
    

---

## 8. Отличие от Open Addressing

|                    | Chaining                      | [[open addressing]]             |
| ------------------ | ----------------------------- | ------------------------------- |
| Хранение           | Linked list в бакете          | Все в массиве бакетов           |
| Удаление           | Легко                         | Сложно, нужна tombstone         |
| Производительность | Хорошо при низкой load factor | Очень хорошо при cache-friendly |
| Аллокации          | Требуются узлы                | Нет лишних аллокаций            |
| Коллизии           | Разрешаются списком           | Разрешаются probing             |

> Swift [[Dictionary]]/[[Set]] выбрали **open addressing**, потому что важна производительность и локальность кэша, а ARC лишние узлы не нужны.

---

## 9. Коллизии в связных списках

Пример коллизий:

```
table size = 5
hash("apple") % 5 = 2
hash("banana") % 5 = 2
hash("cherry") % 5 = 2
```

```
buckets[2] → ("cherry", ...) → ("banana", ...) → ("apple", ...) → nil
```

- Все ключи коллизировали → идут в цепочку
    
- Lookup → перебор списка
    
- В среднем lookup = O(1 + α) где α = load factor
    

---

## 10. Итог

- **Chaining** — классический способ разрешения коллизий
    
- Все коллизии решаются через **связанный список в бакете**
    
- Прост в реализации, но **медленнее и дороже по памяти**
    
- Swift Dictionary/Set использует **open addressing**, но концепция chaining полезна для понимания и сравнения
    

---
