#gcd #concurrency #dispatch #performance #ios #swift #multithreading

---
### Определение

**`DispatchWorkItemFlags`** — это набор опций (option set) в Grand Central Dispatch ([[GCD]]), который позволяет **настроить поведение** [[DispatchWorkItem]] — объекта, представляющего единицу работы, выполняемую асинхронно или синхронно в очереди.

Эти флаги контролируют такие аспекты, как:
- Приоритет выполнения относительно других задач в очереди
- Поведение при отмене задачи
- Управление [[QoS]] (Quality of Service) наследованием
- Работа с барьерами ([[DispatchBarrier|barrier]]s)
- Детектирование зависаний ([[deadlock]]s) в отладочных сборках

---

### Зачем это знать iOS-разработчику?

| Сценарий | Использование флагов |
|---|---|
| **Задачи с высоким приоритетом** | `.highPriority` — ускорить критическую операцию |
| **Задачи, которые не должны блокировать UI** | `.assignCurrentContext` — унаследовать QoS текущего контекста |
| **Защита от случайной отмены** | `.detached` — задача не наследует состояние отмены от родителя |
| **Групповые операции с барьерами** | `.barrier` — монопольный доступ к ресурсу |
| **Отладка инверсии приоритетов** | `.enforceQoS` — предотвращает неявное повышение QoS |

---

### Доступные флаги

| Флаг                        | Описание                                                                                                   | Когда использовать                                                                                          |
| --------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **`.assignCurrentContext`** | Задача наследует QoS текущего контекста (вместо QoS очереди)                                               | Когда нужно выполнить задачу с приоритетом вызвавшего кода                                                  |
| **`.barrier`**              | Задача выполняется как барьер: все предыдущие задачи завершаются до её начала, следующие ждут её окончания | Только для очередей с параллельным ([[concurrent]]) выполнением. Для синхронизации доступа к общим ресурсам |
| **`.detached**              | Задача не наследует состояние отмены от родительской задачи                                                | Для фоновых операций, которые должны выполняться даже если исходная задача отменена                         |
| **`.enforceQoS**            | Принудительно применяет указанный QoS, даже если он выше текущего                                          | Для предотвращения инверсии приоритетов (priority inversion)                                                |
| **`.inheritQoS**            | Задача наследует QoS у очереди (поведение по умолчанию)                                                    | Когда нужно явно указать стандартное поведение                                                              |
| **`.noQoS**                 | Задача не имеет указанного QoS (используется минимальный)                                                  | Для фоновых некритичных операций                                                                            |

---

### Примеры использования

#### 1. **Базовое создание DispatchWorkItem с флагами**

```swift
import Foundation

// Обычный work item (без флагов)
let regularItem = DispatchWorkItem {
    print("Regular task")
}

// Work item с флагами
let highPriorityItem = DispatchWorkItem(flags: .assignCurrentContext) {
    print("High priority task with current context QoS")
}
```

#### 2. **Barrier флаг — синхронизация доступа к ресурсу**

```swift
class ThreadSafeArray<T> {
    private var array: [T] = []
    private let queue = DispatchQueue(label: "com.example.concurrent", attributes: .concurrent)
    
    // Безопасное чтение (не требует барьера)
    func read(at index: Int) -> T? {
        return queue.sync {
            guard index < array.count else { return nil }
            return array[index]
        }
    }
    
    // Запись требует барьера — монопольный доступ
    func append(_ element: T) {
        let workItem = DispatchWorkItem(flags: .barrier) {
            self.array.append(element)
        }
        queue.async(execute: workItem)
    }
    
    // Удаление также требует барьера
    func removeAll() {
        let workItem = DispatchWorkItem(flags: .barrier) {
            self.array.removeAll()
        }
        queue.async(execute: workItem)
    }
}
```

#### 3. **Detached флаг — задача, независимая от отмены родителя**

```swift
class DownloadManager {
    var currentTask: DispatchWorkItem?
    let queue = DispatchQueue.global()
    
    func startDownload() {
        // Основная задача (может быть отменена)
        let mainTask = DispatchWorkItem { [weak self] in
            print("Main download started")
            Thread.sleep(forTimeInterval: 2)
            print("Main download completed")
        }
        
        // Фоновая задача логирования (не должна отменяться вместе с mainTask)
        let logTask = DispatchWorkItem(flags: .detached) {
            print("Logging task started — will continue even if main is cancelled")
            Thread.sleep(forTimeInterval: 5)
            print("Logging completed")
        }
        
        currentTask = mainTask
        
        queue.async(execute: mainTask)
        queue.async(execute: logTask)
    }
    
    func cancel() {
        currentTask?.cancel()
        print("Main download cancelled")
    }
}

let manager = DownloadManager()
manager.startDownload()

DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
    manager.cancel()
}

// Вывод:
// Main download started
// Logging task started — will continue even if main is cancelled
// Main download cancelled
// Main download completed  (не будет, т.к. отменена)
// Logging completed          (будет через 5 секунд, несмотря на отмену)
```

#### 4. **enforceQoS — предотвращение инверсии приоритетов**

```swift
class PriorityInversionDemo {
    let highQueue = DispatchQueue.global(qos: .userInteractive)
    let lowQueue = DispatchQueue.global(qos: .background)
    let mediumQueue = DispatchQueue.global(qos: .utility)
    
    var sharedResource = 0
    let lock = NSLock()
    
    func demonstrateInversion() {
        // Без enforceQoS:
        // Низкоприоритетная задача захватывает ресурс
        lowQueue.async {
            self.lock.lock()
            print("Low priority: Resource locked")
            Thread.sleep(forTimeInterval: 2)
            self.lock.unlock()
            print("Low priority: Resource released")
        }
        
        // Высокоприоритетная задача ждёт ресурс (инверсия приоритетов)
        highQueue.async {
            print("High priority: Waiting for resource...")
            self.lock.lock()
            print("High priority: Resource acquired")
            self.lock.unlock()
        }
    }
    
    func demonstrateWithEnforceQoS() {
        // С enforceQoS высокоприоритетная задача может "унаследовать" приоритет
        let workItem = DispatchWorkItem(flags: .enforceQoS) {
            // Критическая секция
            self.lock.lock()
            print("Critical section with enforced QoS")
            Thread.sleep(forTimeInterval: 0.5)
            self.lock.unlock()
        }
        highQueue.async(execute: workItem)
    }
}
```

#### 5. **assignCurrentContext — наследование QoS вызывающего контекста**

```swift
class NetworkService {
    let backgroundQueue = DispatchQueue.global(qos: .background)
    
    func fetchData(completion: @escaping (Data?) -> Void) {
        // Этот work item получит QoS текущего контекста (.userInitiated),
        // а не .background, как у очереди
        let workItem = DispatchWorkItem(flags: .assignCurrentContext) {
            // Симуляция сетевого запроса
            Thread.sleep(forTimeInterval: 1)
            let data = Data()
            completion(data)
        }
        
        backgroundQueue.async(execute: workItem)
    }
}

// В UI-контексте (.userInteractive)
let service = NetworkService()
service.fetchData { data in
    // completion будет вызван с тем же QoS,
    // что и при вызове fetchData
    print("Data received with current QoS")
}
```

#### 6. **Комбинация нескольких флагов**

```swift
// Можно комбинировать флаги через массив или оператор []
let combinedFlags = DispatchWorkItemItem(flags: [.barrier, .enforceQoS]) {
    // Критическая секция с барьером и принудительным QoS
}

// Или через оператор |
let combinedItem = DispatchWorkItem(flags: .barrier.union(.enforceQoS)) {
    print("Combined barrier and enforceQoS")
}
```

---

### Работа с QoS (Quality of Service)

| QoS класс | Приоритет | Пример использования |
|---|---|---|
| **`.userInteractive`** | Самый высокий | UI-обновления, анимации (должны быть очень быстрыми) |
| **`.userInitiated`** | Высокий | Действия пользователя, которые требуют немедленного результата |
| **`.default`** | Средний | По умолчанию |
| **`.utility`** | Низкий | Длительные задачи с прогресс-баром (загрузка, импорт) |
| **`.background`** | Самый низкий | Фоновые задачи, обслуживание, индексация |

```swift
// Пример: задача с фоновым QoS, но которая должна унаследовать контекст
let backgroundItem = DispatchWorkItem(flags: .assignCurrentContext) {
    // Выполнится с QoS вызвавшего кода, а не .background
}

// Задача, которая принудительно повышает QoS
let enforcedItem = DispatchWorkItem(flags: .enforceQoS) {
    // Критическая работа, требующая высокого приоритета
}
```

---

### DispatchWorkItemFlags и отмена задач

```swift
class CancellableTask {
    let queue = DispatchQueue.global()
    var workItem: DispatchWorkItem?
    
    func start() {
        // Обычный work item (отменяемый)
        let item = DispatchWorkItem { [weak self] in
            for i in 0..<100 {
                // Проверяем, не отменена ли задача
                if item.isCancelled {
                    print("Task cancelled at iteration \(i)")
                    return
                }
                print("Processing \(i)")
                Thread.sleep(forTimeInterval: 0.1)
            }
            print("Task completed")
        }
        
        workItem = item
        queue.async(execute: item)
    }
    
    func cancel() {
        workItem?.cancel()
        print("Cancellation requested")
    }
}

// detached задача не наследует состояние отмены
class DetachedTaskExample {
    let queue = DispatchQueue.global()
    
    func startParentTask() {
        let parentItem = DispatchWorkItem { [weak self] in
            print("Parent task started")
            
            // Эта задача выполнится даже если parentItem будет отменён
            let detachedItem = DispatchWorkItem(flags: .detached) {
                print("Detached task — will continue even if parent is cancelled")
                Thread.sleep(forTimeInterval: 2)
                print("Detached task completed")
            }
            
            self?.queue.async(execute: detachedItem)
            
            // Имитация работы
            for i in 0..<5 {
                if parentItem.isCancelled {
                    print("Parent task cancelled at iteration \(i)")
                    return
                }
                print("Parent iteration \(i)")
                Thread.sleep(forTimeInterval: 0.2)
            }
            print("Parent task completed")
        }
        
        queue.async(execute: parentItem)
        
        // Отменяем родительскую задачу через 0.5 секунды
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            parentItem.cancel()
            print("Parent cancellation requested")
        }
    }
}
```

---

### Сравнение флагов

| Флаг | Наследует QoS | Наследует отмену | Барьер | Приоритет |
|---|---|---|---|---|
| **По умолчанию (без флагов)** | От очереди | Да | Нет | Как у очереди |
| **`.assignCurrentContext`** | От вызвавшего контекста | Да | Нет | Как у вызвавшего |
| **`.detached`** | От очереди | Нет | Нет | Как у очереди |
| **`.barrier`** | От очереди | Да | Да | Как у очереди |
| **`.enforceQoS`** | Принудительный | Да | Нет | Указанный |
| **`.inheritQoS`** | От очереди (явно) | Да | Нет | Как у очереди |
| **`.noQoS`** | Минимальный | Да | Нет | Минимальный |

---

### Лучшие практики

1. **Для синхронизации доступа к разделяемым ресурсам** используйте `.barrier` на concurrent-очереди
2. **Для фоновых задач, которые не должны отменяться** используйте `.detached`
3. **Для UI-задач, вызываемых из фона** используйте `.assignCurrentContext` перед переходом на main queue
4. **Для критических секций с высоким приоритетом** используйте `.enforceQoS`
5. **Не комбинируйте `.barrier` с `.detached`** — это не имеет смысла, так как барьерные задачи обычно не должны отменяться

```swift
// ✅ Правильно: барьер для синхронизации
let barrierItem = DispatchWorkItem(flags: .barrier) {
    // синхронизированный доступ
}

// ✅ Правильно: detached для независимых фоновых задач
let detachedItem = DispatchWorkItem(flags: .detached) {
    // независимая работа
}

// ❌ Неправильно: комбинация, лишённая смысла
let confusedItem = DispatchWorkItem(flags: [.barrier, .detached]) {
    // .detached игнорируется для барьерных задач
}
```

---

### Итог

**`DispatchWorkItemFlags`** в GCD предоставляют тонкую настройку поведения задач:

| Флаг | Основное применение |
|---|---|
| **`.assignCurrentContext`** | Наследование QoS вызвавшего контекста |
| **`.barrier`** | Монопольный доступ к ресурсу (только на concurrent-очереди) |
| **`.detached`** | Независимость от отмены родительской задачи |
| **`.enforceQoS`** | Предотвращение инверсии приоритетов |
| **`.inheritQoS`** | Явное указание наследования от очереди |
| **`.noQoS`** | Минимальный приоритет для фоновых задач |

**Ключевые выводы:**
- Флаги позволяют управлять QoS, барьерами и поведением при отмене
- Используйте `.barrier` для синхронизации доступа к ресурсам
- Используйте `.assignCurrentContext` для задач, вызываемых из UI-контекста
- Используйте `.detached` для независимых фоновых операций
- Помните, что не все флаги можно комбинировать (`barrier` + `detached` не имеют смысла)

Понимание `DispatchWorkItemFlags` необходимо для эффективной и безопасной работы с GCD в многопоточных iOS-приложениях.