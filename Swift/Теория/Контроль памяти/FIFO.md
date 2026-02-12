#memory_control #Swift 
**FIFO** (First-In-First-Out) — принцип «первым пришёл — первым ушёл».  
Элементы добавляются в **конец** очереди (`enqueue`), извлекаются с **начала** (`dequeue`).

В [[Swift]] нет встроенной структуры [[Queue]], но её легко реализовать на базе [[Array]] (самый распространённый и эффективный способ).

### 1. Классическая реализация на Array

```swift
struct Queue<T> {
    private var elements: [T] = []
    
    /// Добавление элемента в конец очереди — O(1)
    mutating func enqueue(_ element: T) {
        elements.append(element)
    }
    
    /// Извлечение элемента из начала очереди — O(n) из-за shift
    mutating func dequeue() -> T? {
        return elements.isEmpty ? nil : elements.removeFirst()
    }
    
    /// Просмотр первого элемента без удаления — O(1)
    func peek() -> T? {
        elements.first
    }
    
    var isEmpty: Bool {
        elements.isEmpty
    }
    
    var count: Int {
        elements.count
    }
}
```

### 2. Оптимизированная версия (O(1) для dequeue)

Используем **два указателя** или **циклический буфер** — самый эффективный способ на `Array`.

```swift
struct EfficientQueue<T> {
    private var elements: [T] = []
    private var head = 0
    
    mutating func enqueue(_ element: T) {
        elements.append(element)
    }
    
    mutating func dequeue() -> T? {
        guard head < elements.count else { return nil }
        defer { head += 1 }
        return elements[head]
    }
    
    func peek() -> T? {
        head < elements.count ? elements[head] : nil
    }
    
    var isEmpty: Bool { head >= elements.count }
    var count: Int { elements.count - head }
    
    /// Очистка старых элементов (опционально, экономит память)
    mutating func clean() {
        if head > 1000 {  // порог можно настроить
            elements.removeFirst(head)
            head = 0
        }
    }
}
```

### 3. Примеры использования

#### Простой пример

```swift
var q = Queue<Int>()
q.enqueue(10)
q.enqueue(20)
q.enqueue(30)

print(q.dequeue())  // 10
print(q.peek())     // 20
print(q.dequeue())  // 20
print(q.dequeue())  // 30
print(q.dequeue())  // nil
```

#### Эффективная очередь в реальном сценарии (очередь задач)

```swift
struct Task {
    let id: Int
    let action: () -> Void
}

var taskQueue = EfficientQueue<Task>()

// Добавляем задачи
taskQueue.enqueue(Task(id: 1) { print("Задача 1 выполнена") })
taskQueue.enqueue(Task(id: 2) { print("Задача 2 выполнена") })

// Асинхронная обработка
DispatchQueue.global().async {
    while let task = taskQueue.dequeue() {
        task.action()
        taskQueue.clean()  // периодическая очистка
    }
}
```

#### Очередь сообщений в чате

```swift
struct ChatMessage {
    let text: String
    let isFromMe: Bool
}

var messageQueue = Queue<ChatMessage>()

// Добавляем входящие сообщения
messageQueue.enqueue(ChatMessage(text: "Привет!", isFromMe: false))
messageQueue.enqueue(ChatMessage(text: "Как дела?", isFromMe: false))

// Отправляем по очереди
while let msg = messageQueue.dequeue() {
    // отправка в UI / сеть
    print("→ \(msg.text)")
}
```

### 4. Сравнение реализаций

| Реализация          | enqueue | dequeue | peek  | Память          | Когда использовать                  |
|---------------------|---------|---------|-------|-----------------|-------------------------------------|
| На `Array` (removeFirst) | O(1)    | O(n)    | O(1)  | O(n)            | Маленькие очереди (< 1000 элементов) |
| EfficientQueue (head)    | O(1)    | O(1)    | O(1)  | O(n), с очисткой | Большие очереди, высокая нагрузка   |
| Двусвязный список   | O(1)    | O(1)    | O(1)  | O(n) + overhead | Если нужна частая вставка в середину |
| Circular Buffer     | O(1)    | O(1)    | O(1)  | Фиксированный   | Фиксированный размер очереди        |

### 5. Короткие рекомендации (2026)

- **Маленькие очереди** (< 1000 элементов) → обычный `Array` + `append` / `removeFirst`
- **Большие / высоконагруженные** → `EfficientQueue` с указателем `head` и периодической `clean()`
- **Фиксированный размер** → кольцевой буфер (Circular Buffer)
- **Очередь задач / сообщений** → `DispatchQueue` или `OperationQueue` (если не нужна строгая FIFO-логика)
- **Для обучения** → реализуй сам `Queue` на `Array` и сравни с `EfficientQueue`

**Главное правило**:  
«Если очередь небольшая — просто используй `Array`.  
Если большая и важна производительность — делай `EfficientQueue` с указателем `head` и очисткой.»
