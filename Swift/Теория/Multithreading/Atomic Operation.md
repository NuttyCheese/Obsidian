### 1. Что такое атомарная операция

**Атомарная операция** — это операция с общей переменной, которая **выполняется как одно неделимое действие**:

- либо полностью завершается  
- либо не выполняется вообще  

Никакой другой поток **не может увидеть промежуточное состояние** во время выполнения атомарной операции.

**Классические примеры атомарных операций**:

- `load()` — атомарное чтение значения  
- `store()` — атомарная запись значения  
- `fetch_add()`, `fetch_sub()` — чтение + арифметика + запись  
- `compare_exchange()` (CAS — Compare-And-Swap) — сравнение и условная замена  
- `exchange()` — атомарная замена значения с возвратом старого  

**Зачем они нужны**:

- **предотвращают [[data race]]** при одновременном доступе нескольких потоков  
- **гарантируют согласованность** даже без дополнительных блокировок  
- **очень быстрые** (обычно 1–2 инструкции процессора)  
- **масштабируются** гораздо лучше, чем mutex/lock

### 2. Атомарные операции в [[Swift]] (2026)

В Swift **нет встроенных** атомарных типов в стандартной библиотеке.

**Основной способ** в 2026 году — использовать официальную библиотеку:
**[swift-atomics](https://github.com/apple/swift-atomics)**

Это **официально поддерживаемая** библиотека от Apple, входящая в экосистему Swift.

```swift
// Добавляем зависимость в Package.swift
.package(url: "https://github.com/apple/swift-atomics.git", from: "1.2.0"),
```

### 3. Основные атомарные типы в swift-atomics

| Тип                                                | Описание                               | Самые полезные операции                                                 | Применение               |
| -------------------------------------------------- | -------------------------------------- | ----------------------------------------------------------------------- | ------------------------ |
| `Atomic<Int>` / `Atomic<Int64>` / `Atomic<UInt64>` | Атомарные целые числа                  | `load()`, `store()`, `fetch_add()`, `fetch_sub()`, `compare_exchange()` | Счётчики, флаги, индексы |
| `Atomic<Bool>`                                     | Атомарный [[Bool]]                     | `load()`, `store()`, `exchange()`                                       | Флаги (вкл/выкл)         |
| `Atomic<UnsafeRawPointer>`                         | Атомарный указатель                    | `load()`, `store()`, `compare_exchange()`                               | Lock-free структуры      |
| `AtomicReference<T>`                               | Атомарная ссылка на объект ([[class]]) | `load()`, `store()`, `compare_exchange()`                               | Lock-free кэш, список    |
| `ManagedAtomic<T>`                                 | Для типов, которые требуют [[ARC]]     | Аналогично Atomic                                                       | Редко                    |

### 4. Самые популярные шаблоны 2026 года

#### Шаблон 1 — Атомарный счётчик (самый частый)

```swift
import Atomics

let counter = Atomic<Int>(0)

// 1000 потоков увеличивают счётчик
for _ in 0..<1000 {
    Task.detached {
        for _ in 0..<1000 {
            counter.wrappingIncrement(by: 1, ordering: .relaxed)
        }
    }
}

// Итог всегда будет ровно 1_000_000
print(counter.load(ordering: .relaxed))  // 1000000
```

**Порядок памяти** (ordering):

- `.relaxed` — самый быстрый, минимальные гарантии  
- `.acquiring` / `.releasing` — для acquire/release семантики  
- `.sequentiallyConsistent` — самые сильные гарантии (самый медленный)

#### Шаблон 2 — CAS (Compare-And-Swap) — основа lock-free структур

```swift
let value = Atomic<Int>(0)

func incrementIfLessThan(target: Int, by amount: Int) {
    var current = value.load(ordering: .relaxed)
    
    while true {
        guard current < target else { return }
        
        if value.compareExchange(
            expected: current,
            desired: current + amount,
            ordering: .acquiringAndReleasing,
            failureOrdering: .relaxed
        ).exchanged {
            return  // успех
        }
        
        // CAS не удался — значение изменилось другим потоком
        current = value.load(ordering: .relaxed)
    }
}
```

#### Шаблон 3 — Lock-free односвязный список (очень мощный пример)

```swift
final class Node<T>: @unchecked Sendable {
    let value: T
    var next: Node? = nil
}

actor LockFreeStack<T> {
    private var head = AtomicReference<Node<T>?>(nil)
    
    func push(_ value: T) {
        let newNode = Node(value: value)
        
        while true {
            let oldHead = head.load(ordering: .relaxed)
            newNode.next = oldHead
            
            if head.compareExchange(
                expected: oldHead,
                desired: newNode,
                ordering: .acquiringAndReleasing,
                failureOrdering: .relaxed
            ).exchanged {
                return
            }
        }
    }
    
    func pop() -> T? {
        while true {
            let oldHead = head.load(ordering: .relaxed)
            guard let oldHead else { return nil }
            
            let newHead = oldHead.next
            
            if head.compareExchange(
                expected: oldHead,
                desired: newHead,
                ordering: .acquiringAndReleasing,
                failureOrdering: .relaxed
            ).exchanged {
                return oldHead.value
            }
        }
    }
}
```

### 5. Типичные ошибки и ловушки 2026 года

| Ошибка                                     | Последствия                            | Как избежать                                               |
| ------------------------------------------ | -------------------------------------- | ---------------------------------------------------------- |
| Забыть указать ordering                    | Undefined behavior / неверные значения | Всегда явно задавать `.relaxed` / `.acquiringAndReleasing` |
| Использовать `Atomic` для сложных объектов | Гонка данных внутри объекта            | Только примитивы или `@unchecked Sendable` + синхронизация |
| Думать, что `Atomic` заменяет actor        | Нет — `Atomic` для примитивов          | Для сложного состояния — actor                             |
| Использовать `Atomic` в UI-потоке          | Ненужный overhead                      | UI — [[@MainActor]], данные — [[actor]] / Atomic           |
| Забыть `load()` / `store()` с ordering     | Undefined behavior                     | Всегда указывать ordering                                  |

### 6. Atomic vs другие механизмы синхронизации (2026 сравнение)

| Механизм                       | Скорость | Сложность | Подходит для                               | Рекомендация 2026            | Когда использовать         |
| ------------------------------ | -------- | --------- | ------------------------------------------ | ---------------------------- | -------------------------- |
| `Atomic<T>` (swift-atomics)    | ★★★★★    | ★★★★☆     | Примитивы ([[Int]], [[Bool]], [[Pointer]]) | Высокопроизводительные места | Счётчики, флаги, lock-free |
| `actor`                        | ★★★★☆    | ★★☆☆☆     | Любое состояние                            | Основной выбор               | Сложное состояние          |
| `os_unfair_lock`               | ★★★★★    | ★★★★☆     | Простые блокировки                         | Низкоуровневый код           | Когда actor избыточен      |
| `@MainActor`                   | ★★★☆☆    | ★★☆☆☆     | UI                                         | Для UI                       | ViewModel, UI              |
| [[DispatchQueue]] ([[serial]]) | ★★★★☆    | ★★☆☆☆     | Legacy                                     | Legacy                       | Старый код                 |

**Вывод 2026**:
- `Atomic` — **самый быстрый** и **низкоуровневый** способ синхронизации примитивов  
- `actor` — **самый удобный** и **безопасный** способ для большинства случаев  
- `os_unfair_lock` — для **максимальной производительности** в очень горячих местах  
- В 2026 году **90%** изменяемого состояния должно жить в `actor`  
- `Atomic` используют **только** для счётчиков, флагов, lock-free структур

### 7. Лучшие практики 2026 года

- **Используй `swift-atomics`** — официальная библиотека Apple  
- **Всегда** указывай `ordering` явно — `.relaxed` для максимальной скорости, `.acquiringAndReleasing` для acquire/release  
- **Не используй `Atomic` для сложных объектов** — только примитивы  
- **Для сложного состояния** — **actor**  
- **Для UI** — **@MainActor**  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит unsafe использование  
- **Тестирование** — Thread Sanitizer + Instruments + stress-тесты  
- **Документируй** — пиши комментарий «Atomic с .acquiringAndReleasing для acquire/release семантики»

**Короткий девиз 2026**:
> «Atomic — это когда ты хочешь **максимальную скорость** и **атомарность** для простых значений.  
> В 2026 году это **лучший выбор** для счётчиков, флагов и lock-free структур.  
> Всё остальное — actor.»
