#data_structures #filo #stack #first_in_last_out #ordering #collection #algorithm #sequence #ios #memory_management
**FILO** (First-In-Last-Out) / **LIFO** (Last-In-First-Out) — принцип «последним пришёл — первым ушёл».  
Элементы добавляются в **вершину** стека (`push`), извлекаются с **вершины** (`pop`).

В [[Swift]] нет встроенного типа `Stack`, но его очень просто и эффективно реализовать на базе [[Array]].

### 1. Классическая и самая популярная реализация

```swift
struct Stack<Element> {
    private var elements: [Element] = []
    
    /// Добавление элемента на вершину — O(1)
    mutating func push(_ element: Element) {
        elements.append(element)
    }
    
    /// Извлечение элемента с вершины — O(1)
    mutating func pop() -> Element? {
        elements.popLast()
    }
    
    /// Просмотр вершины без удаления — O(1)
    func peek() -> Element? {
        elements.last
    }
    
    var isEmpty: Bool {
        elements.isEmpty
    }
    
    var count: Int {
        elements.count
    }
}
```

### 2. Примеры использования

#### Простой пример

```swift
var stack = Stack<Int>()
stack.push(10)
stack.push(20)
stack.push(30)

print(stack.pop())    // 30
print(stack.peek())   // 20
print(stack.pop())    // 20
print(stack.pop())    // 10
print(stack.pop())    // nil
```

#### Реальный сценарий: история навигации (back stack)

```swift
struct NavigationState {
    let screen: String
    let data: Any?
}

var backStack = Stack<NavigationState>()

// Пользователь переходит на новый экран
backStack.push(NavigationState(screen: "Home", data: nil))
backStack.push(NavigationState(screen: "Profile", data: user))
backStack.push(NavigationState(screen: "Settings", data: nil))

// Нажата кнопка "Назад"
if let previous = backStack.pop() {
    print("Вернуться на: \(previous.screen)")
    // обновить UI на previous.screen
}
```

#### Стек вызовов в рекурсии (концептуально)

```swift
func factorial(n: Int) -> Int {
    var stack = Stack<Int>()
    stack.push(n)
    
    var result = 1
    while let current = stack.pop() {
        if current > 1 {
            stack.push(current - 1)
        }
        result *= current
    }
    return result
}
```

### 3. Преимущества реализации на `Array`

| Операция   | Сложность | Почему быстро |
|------------|-----------|---------------|
| `push`     | O(1)      | `append` амортизировано O(1) |
| `pop`      | O(1)      | `popLast` — просто уменьшить count |
| `peek`     | O(1)      | доступ к `last` |
| `isEmpty`  | O(1)      | проверка `count == 0` |

**Альтернативы (редко нужны)**

- **Linked List** — если нужна частая вставка/удаление в середине (O(n) в Array)
- **Deque** (из Swift Collections) — если нужна двухсторонняя очередь
- **Circular Buffer** — если нужен фиксированный размер

### 4. Короткие рекомендации (2026)

- **Маленькие и средние стеки** → используй `Array` + `append` / `popLast`
- **Очень большие стеки** (> 100 000 элементов) → добавляй `reserveCapacity` заранее
- **Для обучения** → пиши `Stack<T>` на `Array` — это классика
- **В production** → если нужна строгая очередь задач → лучше [[DispatchQueue]] или [[OperationQueue]]
- **Для истории навигации** → часто достаточно `Array` + `removeLast()` / `last`

**Главное правило**:
> «Стек в Swift = `Array` с операциями `append` и `popLast`.  
> Всё остальное — преждевременная оптимизация.»
