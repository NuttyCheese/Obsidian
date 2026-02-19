**`deinit`** — это специальный метод в классах [[Swift]], который **автоматически вызывается** системой **непосредственно перед тем**, как объект будет удалён из памяти (после того, как ARC обнулил последнюю сильную ссылку на него).

Это **единственный** способ в Swift гарантированно выполнить финальную очистку ресурсов именно в момент уничтожения объекта.

### 1. Ключевые факты о deinit (2026 актуально)

| Характеристика                      | Значение / Особенность                                    | Важные детали                            |
| ----------------------------------- | --------------------------------------------------------- | ---------------------------------------- |
| Где можно использовать              | **Только в классах** (class)                              | struct и enum не имеют deinit            |
| Сколько раз вызывается              | **Ровно один раз** на экземпляр                           | Даже если объект создавался много раз    |
| Параметры и возвращаемое значение   | Нет — `deinit { ... }` без скобок и [[return]]            | Просто тело блока                        |
| Когда точно вызывается              | Когда [[ARC]] обнуляет последнюю сильную ссылку           | После вызова deinit память освобождается |
| Что НЕ вызывается deinit            | При [[retain cycle]] (замкнутый цикл сильных ссылок)      | Самая частая причина утечек памяти       |
| Можно ли вызвать вручную            | Нет — `deinit` запрещено вызывать вручную                 | Компилятор выдаст ошибку                 |
| Можно ли переопределить в подклассе | Да — `override deinit { ... }`                            | Вызывается после deinit родителя         |
| Связь с concurrency                 | Вызывается на том же потоке/акторе, где обнулилась ссылка | В Swift 6 строже проверяется             |

### 2. Самые важные и рекомендуемые паттерны использования deinit в 2026 году

#### Паттерн 1: Логирование жизненного цикла (отладка)

```swift
class ProfileViewModel {
    init() {
        print("ProfileViewModel создан")
    }
    
    deinit {
        print("ProfileViewModel уничтожен")
    }
}
```

Очень полезно при поиске утечек памяти — если deinit не вызывается → [[retain cycle]].

#### Паттерн 2: Отмена таймеров и задач (самый частый в [[UIKit]])

```swift
class TimerViewController: UIViewController {
    private var timer: Timer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.updateTimer()
        }
    }
    
    deinit {
        timer?.invalidate()
        print("Таймер остановлен при уничтожении контроллера")
    }
}
```

Без deinit таймер будет тикать даже после ухода с экрана → retain cycle.

#### Паттерн 3: Отписка от уведомлений / KVO

```swift
class NotificationObserver {
    init() {
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(handleNotification),
                                               name: .UIApplicationDidBecomeActive,
                                               object: nil)
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
        print("Отписка от уведомлений завершена")
    }
    
    @objc func handleNotification() { ... }
}
```

#### Паттерн 4: Закрытие файлов / соединений / потоков

```swift
class FileProcessor {
    private var fileHandle: FileHandle?
    
    init(path: String) throws {
        fileHandle = try FileHandle(forWritingTo: URL(fileURLWithPath: path))
    }
    
    deinit {
        try? fileHandle?.close()
        print("Файл закрыт при уничтожении объекта")
    }
}
```

### 3. Самые опасные ловушки retain cycle (и как их ловить через deinit)

```swift
// Классический retain cycle (deinit НЕ вызовется)
class Parent {
    var child: Child?
    deinit { print("Parent deinit") }
}

class Child {
    var parent: Parent?          // сильная ссылка!
    deinit { print("Child deinit") }
}

var p: Parent? = Parent()
var c = Child()
p?.child = c
c.parent = p
p = nil
c = nil
// НИЧЕГО НЕ ВЫВЕДЕТСЯ — объекты живы вечно
```

Решение — **[[weak]]** или **[[unowned]]**:

```swift
class Child {
    weak var parent: Parent?     // или unowned, если уверен в жизненном цикле
    deinit { print("Child deinit") }
}
```

Теперь deinit вызовется.

### 4. Лучшие практики deinit в Swift 2026

- **Всегда** добавляй `deinit { print(...) }` в классы, которые держат ресурсы (таймеры, observers, файлы, сетевые задачи)  
- **Пиши deinit в самом конце класса** — сразу видно, что происходит при уничтожении  
- **Не выполняй тяжёлую работу в deinit** — только быструю очистку (invalidate, removeObserver, close)  
- **Не полагайся на deinit для критической логики** — это не finally из других языков  
- **В SwiftUI / Combine** — используй `.onDisappear` или `.task {}` вместо deinit  
- **Swift 6 strict concurrency** — deinit вызывается на том же акторе, где обнулилась последняя ссылка  
- **Документируйте** — пиши комментарий «deinit — отмена таймера и отписка от уведомлений»

**Короткий девиз 2026**:
> `deinit` — это **последний шанс** класса сказать «прощай» перед уходом из памяти.  
> В 2026 году используй его **только** для очистки ресурсов: invalidate таймеров, removeObserver, close файлов/соединений.  
> Если deinit не вызывается → 99% случаев это retain cycle.  
> Пиши `deinit` в каждом классе, который держит что-то внешнее — это спасёт от утечек.
