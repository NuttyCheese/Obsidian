**Связанный список (Linked List)** в Swift — это классическая линейная структура данных, где элементы (узлы) связаны между собой ссылками, а не хранятся в непрерывном блоке памяти (как в массиве).

В 2025–2026 годах связные списки в реальных iOS-приложениях используются **очень редко** как самостоятельная структура (Array почти всегда эффективнее).  
Но они остаются **важной темой** для собеседований, алгоритмических задач и понимания основ структур данных.

### 1. Основные виды связных списков в Swift

| Вид списка              | Особенности хранения                          | Сложность операций                          | Когда реально используют в 2026 |
|-------------------------|-----------------------------------------------|---------------------------------------------|----------------------------------|
| Односвязный (Singly Linked) | каждый узел → только `next`                   | Доступ: O(n), вставка/удаление в начало: O(1) | Алгоритмические задачи, LRU Cache (иногда) |
| Двусвязный (Doubly Linked)  | каждый узел → `next` и `prev`                 | Доступ: O(n), вставка/удаление: O(1) при наличии указателя | Реализация Deque, LRU Cache, Undo/Redo |
| Циклический (Circular)      | последний узел → на первый                    | —                                           | Кольцевые буферы, планировщики задач (редко) |
| С заглушкой (Sentinel / Dummy nodes) | фиктивные узлы в начале и/или конце          | Упрощает код вставки/удаления               | Очень часто в задачах LeetCode |

### 2. Классическая реализация односвязного списка (2026 стиль)

```swift
// Узел списка
class ListNode<T> {
    var value: T
    var next: ListNode<T>?
    
    init(_ value: T) {
        self.value = value
    }
}

// Сам список (с head и tail для O(1) append)
final class LinkedList<T> {
    private(set) var head: ListNode<T>?
    private(set) var tail: ListNode<T>?
    private(set) var count = 0
    
    var isEmpty: Bool { head == nil }
    
    // O(1) добавление в конец
    func append(_ value: T) {
        let newNode = ListNode(value)
        
        if let tail {
            tail.next = newNode
            self.tail = newNode
        } else {
            head = newNode
            tail = newNode
        }
        
        count += 1
    }
    
    // O(1) добавление в начало
    func prepend(_ value: T) {
        let newNode = ListNode(value)
        newNode.next = head
        head = newNode
        
        if tail == nil {
            tail = newNode
        }
        
        count += 1
    }
    
    // O(n) удаление по значению (можно улучшить, если хранить индексы)
    func remove(_ value: T) where T: Equatable {
        guard var current = head else { return }
        
        if current.value == value {
            head = current.next
            if head == nil { tail = nil }
            count -= 1
            return
        }
        
        while let next = current.next {
            if next.value == value {
                current.next = next.next
                if next.next == nil { tail = current }
                count -= 1
                return
            }
            current = next
        }
    }
    
    // O(1) очистка
    func removeAll() {
        head = nil
        tail = nil
        count = 0
    }
}
```

### 3. Сравнение LinkedList vs Array (реальность 2026)

| Операция                  | Array (Contiguous)      | LinkedList (Singly)     | Победитель в реальных приложениях |
|---------------------------|--------------------------|--------------------------|------------------------------------|
| Доступ по индексу         | O(1)                     | O(n)                     | Array                              |
| Добавление в конец        | O(1) амортизировано      | O(1) с tail              | Ничья (Array часто быстрее)        |
| Добавление в начало       | O(n) (сдвиг)             | O(1)                     | LinkedList (но редко нужно)        |
| Удаление с начала         | O(n)                     | O(1)                     | LinkedList                         |
| Удаление по значению      | O(n)                     | O(n)                     | Ничья                              |
| Память                    | непрерывная, компактная  | разрозненная + overhead  | Array (лучше кэш-память)           |
| Итерация                  | очень быстрая            | медленнее (не кэш-френдли) | Array                            |

**Вывод**:  
В реальных iOS-приложениях 2025–2026 LinkedList почти никогда не используется вместо `Array`.  
Основные места, где LinkedList встречается:

- алгоритмические задачи (LeetCode, собеседования)  
- реализация LRU Cache (часто именно двусвязный список + словарь)  
- Undo/Redo стеки (очень редко)  
- некоторые низкоуровневые структуры в Core Foundation / Metal (редко)

### 4. Лучшие практики при работе со связными списками в Swift 2026

- **Никогда** не реализуй LinkedList вручную для хранения данных в приложении — используй `Array`  
- **Используй** `ListNode` только в задачах на алгоритмы  
- **Для LRU Cache** — стандартная реализация в Swift 5.5+ это `LinkedHashMap` / `NSCache` + двусвязный список  
- **В production** чаще используют `Deque` из swift-collections (двусвязный список под капотом)  
- **Swift 6 strict concurrency** — если делаешь свой LinkedList, делай его `actor` или `Sendable` при необходимости  
- **Документируйте** — пиши комментарий «Singly Linked List — используется только для алгоритмической задачи»

**Короткий девиз 2026**:
> Linked List — это **учебная** структура данных: отлична для понимания указателей, но в реальном iOS-приложении почти всегда проигрывает `Array`.  
> Используй её только на собеседованиях, в LeetCode и для LRU Cache.  
> В production → `Array`, `Deque`, `NSCache`, `OrderedDictionary`.

Удачи на собеседованиях и в алгоритмических задачах! 🔗