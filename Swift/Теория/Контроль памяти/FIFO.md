#fifo #first-in-first-out #queue #data-structures #swift #algorithms #performance #collections #thread-safety #concurrency

---
**FIFO (First-In-First-Out)**  
(очередь «первым пришёл — первым ушёл» / принцип очереди)

FIFO — это один из самых фундаментальных и часто используемых принципов организации данных в программировании вообще и в [[Swift]]/[[iOS]] в частности.

**Короткое определение:**  
FIFO — это правило, по которому **первым вошёл в структуру данных → первым и выйдет**.  
Представьте очередь в магазине: кто первый встал в очередь — тот первый и будет обслужен.
 
FIFO — это дисциплина обслуживания (scheduling discipline), при которой элементы обрабатываются в том порядке, в котором они поступили.  
В терминах структур данных — это классическая **очередь** ([[Queue]]).

FIFO — это линейная структура данных с двумя основными операциями:
- **Enqueue** (добавление в конец) — O(1) в амортизированном случае
- **Dequeue** (удаление с начала) — O(1) в амортизированном случае

В [[Swift]] наиболее распространённые реализации FIFO — это:

1. `Array` как очередь (самый частый и опасный выбор)
2. `ArraySlice` (редко)
3. `Deque` из swift-collections (рекомендуемый в 2025–2026)
4. `LinkedList` (редко, но иногда полезно)
5. `DispatchQueue` (для concurrency)

### 1. Почему FIFO важен в iOS-разработке

| Сценарий в iOS/UIKit/SwiftUI                  | Почему FIFO критичен здесь                                                 | Типичная реализация в 2026                   |
| --------------------------------------------- | -------------------------------------------------------------------------- | -------------------------------------------- |
| Очередь задач / операций ([[OperationQueue]]) | Задачи выполняются в порядке поступления                                   | `OperationQueue` (FIFO по умолчанию)         |
| Асинхронные операции ([[GCD]])                | `DispatchQueue` — строгая FIFO-очередь                                     | `DispatchQueue.global()` / `serial`          |
| Буфер сообщений в чате / ленте                | Сообщения должны отображаться в порядке получения                          | `Deque` или `Array` + append/removeFirst     |
| Undo/Redo стек                                | Последнее действие — первое отменяется ([[FILO\|LIFO]]), но история — FIFO | `Array` как история + `Array` как redo-stack |
| Очередь анимаций / transition                 | Анимации должны запускаться по порядку поступления                         | `DispatchQueue.main.async` или custom queue  |
| Очередь сетевых запросов                      | Запросы на отправку/получение в порядке вызова                             | `URLSession` + custom queue                  |
| Rate limiting / throttling                    | Запросы отправляются строго по очереди с задержкой                         | `DispatchQueue` + `asyncAfter`               |
| Message queue в многопоточных системах        | Строгий порядок обработки сообщений                                        | `OSAllocatedUnfairLock` + custom FIFO        |

### 2. Реализации FIFO в Swift 2026 года (от худшей к лучшей)

#### Вариант 1. Array как очередь (самый частый, но опасный)

```swift
var queue: [String] = []

func enqueue(_ item: String) {
    queue.append(item)          // O(1) амортизировано
}

func dequeue() -> String? {
    if !queue.isEmpty {
        return queue.removeFirst()  // ← O(n)! Самая большая проблема
    }
    return nil
}
```

**Проблема:**  
`removeFirst()` на `Array` — это **O(n)** операция, потому что все элементы сдвигаются влево.  
При 1000+ элементов и частых dequeue → заметное торможение (особенно на [[main thread]]).

#### Вариант 2. Array + индекс начала (оптимизация для FIFO)

```swift
struct Queue<T> {
    private var elements: [T] = []
    private var head = 0
    
    mutating func enqueue(_ item: T) {
        elements.append(item)
    }
    
    mutating func dequeue() -> T? {
        guard head < elements.count else { return nil }
        defer { head += 1 }
        return elements[head]
    }
    
    var isEmpty: Bool { head >= elements.count }
    
    mutating func clear() {
        elements.removeAll()
        head = 0
    }
}
```

**Плюсы:**  
- `enqueue` и `dequeue` — O(1)  
- Не нужно каждый раз сдвигать массив

**Минусы:**  
- Память растёт бесконечно, если не чистить периодически  
- Нужно вручную вызывать `clear()` или реализовать авто-очистку

#### Вариант 3. Лучший выбор 2025–2026 — swift-collections Deque

```swift
import DequeModule  // SPM: https://github.com/apple/swift-collections

var queue = Deque<String>()

queue.append("Task 1")     // enqueue back
queue.append("Task 2")

let first = queue.popFirst()   // dequeue front — O(1)
let last  = queue.popLast()    // можно и с конца — тоже O(1)

queue.prepend("Urgent task")   // добавить в начало — O(1)
```

**Почему Deque — лучший выбор в 2026:**

- Все операции (append, prepend, popFirst, popLast) — **O(1) амортизировано**
- Нет сдвигов элементов
- Не растёт бесконечно (автоматически перераспределяет память)
- Thread-safe при использовании в [[GCD]]/[[actor]]s (но не атомарный — нужен [[NSLock|lock]] при многопоточности)
- Официально от Apple (swift-collections)

**Рекомендация 2026:**  
В 90% случаев, где нужна FIFO-очередь, используйте именно `Deque<T>` из swift-collections.

### 3. Thread-safety и FIFO в многопоточной среде

FIFO сама по себе **не гарантирует thread-safety**.

```swift
// Ошибка №1: несколько потоков пишут/читают одну очередь
let queue = Deque<String>()

DispatchQueue.global().async {
    queue.append("Task from background")
}

DispatchQueue.main.async {
    print(queue.popFirst() ?? "empty")  // ← race condition!
}
```

**Правильные варианты в 2026:**

1. **[[Serial]] [[DispatchQueue]]** (классика)

```swift
let serialQueue = DispatchQueue(label: "com.app.task-queue")
let dataQueue = Deque<String>()

func enqueue(_ item: String) {
    serialQueue.async {
        dataQueue.append(item)
    }
}

func dequeue() -> String? {
    serialQueue.sync {
        dataQueue.popFirst()
    }
}
```

2. **Actor** (современный и рекомендуемый)

```swift
actor TaskQueue<T> {
    private var queue = Deque<T>()
    
    func enqueue(_ item: T) {
        queue.append(item)
    }
    
    func dequeue() -> T? {
        queue.popFirst()
    }
    
    var isEmpty: Bool { queue.isEmpty }
}

let taskQueue = TaskQueue<String>()

await taskQueue.enqueue("Task 1")
let item = await taskQueue.dequeue()
```

3. **OSAllocatedUnfairLock** (максимальная производительность, если не нужен actor)

```swift
let lock = OSAllocatedUnfairLock()
var queue = Deque<String>()

func enqueue(_ item: String) {
    lock.lock()
    queue.append(item)
    lock.unlock()
}
```

### 4. Типичные ошибки с FIFO в iOS-разработке

1. Использование `Array.removeFirst()` в горячем цикле → O(n²) в худшем случае  
2. Забывать `resume()` у `URLSessionTask` (тоже FIFO-очередь задач)  
3. Не отменять задачи при `deinit` / `viewWillDisappear` → утечки  
4. Использование `DispatchQueue.global()` как FIFO без сериализации → race conditions  
5. Хранение `any` в очереди задач → лишний existential container overhead

### Итоговая рекомендация 2026 года

| Ситуация                                  | Рекомендуемая структура FIFO                     | Почему |
|-------------------------------------------|---------------------------------------------------|--------|
| Обычная очередь задач / сообщений         | `Deque<T>` из swift-collections                   | O(1) все операции, надёжно |
| Многопоточная очередь                     | `actor TaskQueue<T>`                              | Thread-safety без lock’ов |
| Высокопроизводительная очередь            | `Deque<T>` + `OSAllocatedUnfairLock`              | Максимальная скорость |
| Очень простая очередь (до 100 элементов)  | `Array` + индекс начала                           | Достаточно быстро, нет зависимостей |
| Legacy-код / поддержка старых версий      | `Array` + removeFirst()                           | Работает везде, но медленно |

**Коротко и жёстко (2026):**  
FIFO = очередь.  
В 90% случаев используй `Deque<T>` из swift-collections.  
В многопотоке — оберни в `actor`.  
Никогда не делай `removeFirst()` на `Array` в цикле — это O(n²) и убивает производительность.
