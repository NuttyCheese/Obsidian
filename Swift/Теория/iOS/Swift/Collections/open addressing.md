**Open Addressing** (открытая адресация) — это метод разрешения коллизий в хеш-таблицах, при котором **все элементы хранятся непосредственно в массиве бакетов**, а при коллизии ищется **следующая свободная ячейка** по определённому правилу (probing).

Это **основной метод**, который использует **стандартный [[Dictionary]]** и **[[Set Collection]]** в [[Swift]] начиная с Swift 4+ (и по сей день в 2026 году).

### Почему Swift выбрал именно Open Addressing (а не Chaining)

| Аспект                         | Open Addressing (Swift Dictionary/Set)        | Chaining (связные списки в бакетах)      | Победитель в Swift |
| ------------------------------ | --------------------------------------------- | ---------------------------------------- | ------------------ |
| Кэш-дружественность            | Очень высокая (всё в одном массиве)           | Низкая (разбросанные узлы в [[heap]])    | Open Addressing    |
| Количество аллокаций           | Только один массив + ресайз                   | Много маленьких узлов ([[ARC]] overhead) | Open Addressing    |
| Средняя скорость lookup/insert | O(1) при хорошем hash и load factor < 0.7–0.8 | O(1) + overhead на списки                | Open Addressing    |
| Удаление                       | Сложнее (нужны tombstones)                    | Просто (удаляем из списка)               | Chaining           |
| Память при низкой загрузке     | Меньше overhead                               | Больше (каждый узел — объект)            | Open Addressing    |
| Простота реализации            | Сложнее                                       | Проще                                    | [[Chaining]]       |

**Вывод Apple**:  
Open Addressing даёт **лучшую производительность** на реальных нагрузках iOS/macOS благодаря отличной **cache locality** и меньшему количеству аллокаций.  
Chaining чаще используется в Java, Python, Ruby.

### Основные стратегии probing (поиск свободной ячейки)

| Стратегия              | Формула поиска следующего индекса                  | Плюсы                              | Минусы                              | Используется в Swift? |
|------------------------|-----------------------------------------------------|------------------------------------|-------------------------------------|-----------------------|
| Linear Probing         | `(hash + i) % size`                                 | Очень просто, хорошая кэш-локальность | Кластеры (primary clustering)       | **Да** (основной)     |
| Quadratic Probing      | `(hash + i²) % size` или `(hash + i + i²) % size`   | Меньше кластеров                   | Вторичные кластеры, сложнее         | Нет                   |
| Double Hashing         | `(hash1(key) + i × hash2(key)) % size`              | Отличное распределение             | Дороже вычисления                   | Нет                   |

**Swift использует Linear Probing + tombstones + quadratic-подобные оптимизации** (внутренняя реализация сложнее, чем чистый linear).

### Как работает удаление (tombstones)

При удалении элемента **нельзя просто поставить nil**, иначе поиск других коллидирующих элементов может прерваться.

Решение — **[[tombstone]]** (надгробие):

- При удалении: ставим специальный маркер (deleted)
- При поиске: tombstone считается «занято» — продолжаем probing
- При вставке: можно перезаписать tombstone новым элементом

```text
До удаления:
bucket: [k1, k2, k3, nil, nil]   (k2 и k3 коллидировали → попали в 1 и 2)

Удаляем k1:
bucket: [deleted, k2, k3, nil, nil]

Поиск k3:
начинаем с hash(k3)=1 → deleted → продолжаем → 2 → k3 → нашли
```

Без tombstones поиск k3 бы остановился на позиции 1.

### Упрощённая реализация (для понимания)

```swift
struct SimpleHashTable<Key: Hashable, Value> {
    private var buckets: [(key: Key, value: Value)?]
    private var count = 0
    private let loadFactorThreshold = 0.7
    
    init(capacity: Int = 8) {
        buckets = Array(repeating: nil, count: capacity)
    }
    
    private func index(for key: Key) -> Int {
        return abs(key.hashValue) % buckets.count
    }
    
    mutating func insert(_ key: Key, value: Value) {
        var idx = index(for: key)
        var firstDeleted: Int?
        
        while let bucket = buckets[idx] {
            if bucket.key == key {
                buckets[idx] = (key, value) // обновляем
                return
            }
            if bucket.key == nil { // tombstone или пусто
                if firstDeleted == nil { firstDeleted = idx }
            }
            idx = (idx + 1) % buckets.count
        }
        
        // Вставляем в первую найденную tombstone или пустую ячейку
        let insertIdx = firstDeleted ?? idx
        buckets[insertIdx] = (key, value)
        count += 1
        
        // Ресайз при высокой загрузке
        if Double(count) / Double(buckets.count) > loadFactorThreshold {
            resize()
        }
    }
    
    // ... (get, remove, resize опущены для краткости)
}
```

### 5. Как Swift Dictionary использует Open Addressing (внутренности 2026)

- Массив бакетов хранит **кортежи** `(key, value)` + метаданные (occupied / deleted / empty)
- **Linear Probing** с оптимизациями (не чистый +1, а более сложная последовательность)
- **Tombstones** при удалении
- **Ресайз** происходит при load factor ≈ 0.75–0.8 (удваивает размер, rehash всех элементов)
- **Ключи** хранятся **strong**, значения — **[[strong]]**
- **Хороший hash** критичен — если hash плохой → все коллизии → O(n)

### 6. Советы Swift-разработчику

1. **Делай хорошую реализацию Hashable**  
   → `==` и `hash(into:)` должны быть согласованы  
   → не используй изменяемые свойства в `hash(into:)`

2. **Не бойся удалять ключи**  
   Swift Dictionary сам обрабатывает tombstones

3. **Не пытайся реализовать Dictionary вручную**  
   → встроенная реализация очень оптимизирована

4. **Для очень больших данных** — рассмотри `OrderedDictionary` из swift-collections (если нужен порядок)

**Короткий итог 2026**:
> Open Addressing — это когда коллизии разрешаются **внутри массива бакетов**, без списков.  
> Именно так работает **Dictionary** и **Set** в Swift.  
> Ключ к скорости:  
> - хороший hash  
> - низкий load factor  
> - tombstones при удалении  
> - ресайз при росте  
