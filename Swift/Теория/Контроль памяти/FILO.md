#filo #lifo #first-in-last-out #last-in-first-out #stack #data-structures #swift #algorithms #performance #collections #concurrency #thread-safety #call-stack #undo-redo

---
**FILO / LIFO**  
(First-In-Last-Out / Last-In-First-Out)  
«первым пришёл — последним ушёл» / «последним пришёл — первым ушёл»

FILO и LIFO — это **одно и то же**.  
В программировании и в разговорной речи чаще говорят **LIFO** (Last-In-First-Out), а FILO используют реже, но это синонимы.

**Простыми словами:**  
Представьте стопку тарелок.  
- Последняя положенная тарелка (Last In) — первая, которую вы берёте сверху (First Out).  
- Первая положенная тарелка окажется в самом низу и выйдет последней.

Это и есть **стек** ([[Stack]]) — структура данных с принципом LIFO.

### Основные операции стека (LIFO)

| Операция       | Название       | Сложность | Что делает                              | Пример в Swift |
|----------------|----------------|-----------|-----------------------------------------|----------------|
| Добавить сверху | push / append  | O(1)      | Кладём элемент на вершину стека         | `stack.append(x)` |
| Убрать сверху   | pop / removeLast | O(1)   | Берём и удаляем верхний элемент         | `stack.popLast()` |
| Посмотреть верх | peek / top     | O(1)      | Смотрим верхний элемент без удаления    | `stack.last`      |
| Пустой?         | isEmpty        | O(1)      | Проверяем, есть ли элементы             | `stack.isEmpty`   |

### Реализации LIFO в Swift 2025–2026

#### 1. Самая частая и опасная — Array как стек

```swift
var stack: [String] = []

stack.append("Экран 1")     // push
stack.append("Экран 2")
stack.append("Экран 3")

print(stack.popLast())      // → "Экран 3" (самый последний)
print(stack.popLast())      // → "Экран 2"
print(stack.popLast())      // → "Экран 1"
```

**Плюсы:**  
- Очень просто  
- `append` и `popLast()` — O(1) амортизировано  
- Работает везде, даже в Swift 5.0

**Минусы:**  
- `removeLast()` и `popLast()` — O(1), но если часто делать `removeFirst()` — будет O(n)  
- Нет защиты от неправильного использования (можно случайно вызвать `removeFirst()`)

#### 2. Рекомендуемая в 2026 — Deque из swift-collections (как стек)

```swift
import DequeModule

var stack = Deque<String>()

stack.append("Экран A")     // push back
stack.append("Экран B")
stack.append("Экран C")

print(stack.popLast())      // → "Экран C" (LIFO)
print(stack.popLast())      // → "Экран B"
print(stack.popLast())      // → "Экран A"

stack.prepend("Срочный")    // можно и в начало (но для чистого стека не используем)
```

**Почему Deque лучше Array для стека в 2026:**

- `append` и `popLast()` — гарантированно O(1)
- `prepend` и `popFirst()` — тоже O(1) (на случай, если потом понадобится [[FIFO]])
- Автоматическое управление памятью (не растёт бесконечно)
- Официальная библиотека от Apple

**Рекомендация 2026:**  
В любом новом коде, где нужен стек (LIFO), используйте `Deque<T>` и работайте только с концом: `append` + `popLast`.

#### 3. Самый безопасный — кастомный стек с типобезопасностью

```swift
struct Stack<T> {
    private var storage = Deque<T>()
    
    mutating func push(_ item: T) {
        storage.append(item)
    }
    
    mutating func pop() -> T? {
        storage.popLast()
    }
    
    func peek() -> T? {
        storage.last
    }
    
    var isEmpty: Bool { storage.isEmpty }
    
    var count: Int { storage.count }
}

// Использование
var navigationStack = Stack<String>()
navigationStack.push("Home")
navigationStack.push("Profile")
print(navigationStack.pop())     // → "Profile"
print(navigationStack.peek())    // → "Home"
```

**Преимущества:**
- Названия методов понятные (push/pop/peek)
- Невозможно случайно вызвать `popFirst()`
- Легко добавить логирование, лимит размера, undo-историю

### Типичные применения LIFO в [[iOS]]-разработке

1. **Навигационный стек**  
   `UINavigationController` — это классический LIFO: push → добавляем сверху, pop → убираем сверху.

2. **Undo / Redo**  
   ```swift
   var undoStack = Stack<Action>()
   var redoStack = Stack<Action>()
   
   func perform(_ action: Action) {
       undoStack.push(action)
       redoStack = Stack() // очистка redo при новом действии
   }
   
   func undo() {
       if let action = undoStack.pop() {
           action.undo()
           redoStack.push(action)
       }
   }
   ```

3. **Вызовы функций и стек вызовов**  
   Сам рантайм использует аппаратный стек вызовов (call stack) — чистый LIFO.

4. **Парсинг выражений** (Shunting-yard algorithm, вычисление обратной польской нотации)  
   Стек операторов и операндов — классический LIFO.

5. **Обход дерева в глубину (DFS)**  
   Рекурсия или явный стек — оба LIFO.

6. **Браузерная история назад/вперёд**  
   Два стека: назад (LIFO) и вперёд (LIFO после очистки).

### Типичные ошибки с LIFO в iOS

1. Использование `Array.removeFirst()` вместо `popLast()` — O(n) вместо O(1)  
2. Не очищать redo-стек после нового действия — нарушается логика undo/redo  
3. Хранение `any` в стеке задач → лишний existential container overhead  
4. Забывать проверять `isEmpty` перед `pop()` → crash  
5. Использовать `Array` как стек в горячем цикле без оптимизаций

### Рекомендация 2026 года

| Ситуация                                | Лучшая реализация LIFO в 2026                            | Почему                          |
| --------------------------------------- | -------------------------------------------------------- | ------------------------------- |
| Простой стек вызовов / навигация        | `Deque<T>` + `append` / `popLast`                        | O(1), надёжно                   |
| Undo/Redo                               | Два `Deque<Action>` или кастомный `Stack<Action>`        | Чистый [[API]]                  |
| Высоконагруженный стек (10k+ элементов) | `Deque<T>` + `OSAllocatedUnfairLock` (если многопоточно) | Максимальная производительность |
| Очень маленький стек (< 100 элементов)  | `Array` + `append` / `popLast`                           | Достаточно, нет зависимостей    |
| Обучение / учебный проект               | Собственный `struct Stack<T>` на базе `Deque`            | Понятный API, легко расширять   |

**Коротко и жёстко (2026):**
LIFO = стек.  
В 90% случаев — используй `Deque<T>` и работай только с концом (`append` + `popLast`).  
Не делай `removeFirst()` на [[Array]] в цикле — это O(n²) и убивает производительность.  
[[Actor]] или [[NSLock|lock]] — если нужна многопоточность.
