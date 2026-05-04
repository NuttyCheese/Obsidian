#dealloc #swift #memory-management #arc #weak #unowned #retain-cycle #reference-counting #lifecycle #objective-c-runtime #performance #debugging

---
**dealloc**  
(метод освобождения памяти / деструктор)

**dealloc** — это специальный метод в [[Objective-C]] [[Runtime]], который вызывается автоматически **непосредственно перед тем, как объект будет полностью освобождён из памяти**.

В чистом [[Swift]] с [[ARC]] (Automatic Reference Counting) метод `dealloc()` **не нужно** и **нельзя** переопределять в большинстве случаев — компилятор сам генерирует его вызов.  
Но он остаётся важным инструментом для понимания жизненного цикла объектов и отладки утечек памяти.

### Junior level: Что происходит, когда объект "умирает"

1. Счётчик ссылок ([[retain count]]) объекта падает до **0**.
2. ARC автоматически вызывает `dealloc()` на этом объекте.
3. Внутри `dealloc()`:
   - освобождаются все ресурсы (файлы, таймеры, observers, observers [[KVO]], [[CADisplayLink]] и т.д.)
   - вызывается `super.dealloc()` (в Obj-C)
   - после этого память объекта возвращается в кучу
4. Объект больше **никогда** не будет использован — все ссылки на него считаются dangling pointers (висячими указателями).

**Самый простой пример (junior):**

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) создан")
    }
    
    deinit {  // ← это и есть dealloc в Swift
        print("\(name) уничтожен")
    }
}

var p: Person? = Person(name: "Алексей")   // → "Алексей создан"
p = nil                                     // → "Алексей уничтожен"
```

**Правило junior:**  
Если в `deinit` ничего не выводится → объект **не был освобождён** → утечка памяти (retain cycle).

### Middle level: Когда и почему deinit не вызывается

Самые частые причины, почему `deinit` **не срабатывает** (retain cycle):

1. **Сильная ссылка ([[strong]] reference cycle)**

```swift
class Parent {
    var child: Child?
}

class Child {
    var parent: Parent?         // ← strong → cycle
}

let p = Parent()
let c = Child()
p.child = c
c.parent = p

// p и c никогда не будут освобождены → deinit не вызовется
```

Решение — [[weak]] или [[unowned]]:

```swift
class Child {
    weak var parent: Parent?     // ← weak разрывает цикл
    // или unowned (если точно знаешь, что parent живёт дольше)
}
```

2. **Замыкания ([[closure]]s) удерживают [[self]]**

```swift
class TimerController {
    var timer: Timer?
    
    init() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            print("Тик")  // ← замыкание удерживает self strongly
        }
    }
    
    deinit {
        timer?.invalidate()
        print("TimerController уничтожен")  // ← никогда не выведется
    }
}
```

Решение — `[weak self]`:

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    guard let self else { return }
    print("Тик")
}
```

3. **[[KVO]] / [[NotificationCenter]] / [[CADisplayLink]]** без удаления наблюдателя

```swift
NotificationCenter.default.addObserver(self, selector: #selector(handle), name: .someNotification, object: nil)
// без removeObserver в deinit → retain cycle
```

Решение:

```swift
deinit {
    NotificationCenter.default.removeObserver(self)
}
```

### Senior level: Что происходит внутри dealloc (рантайм, [[ARC]], [[Swift]] vs Obj-C)

**В Objective-C (классика):**

```objc
- (void)dealloc {
    // 1. Освобождаются ivars (instance variables)
    // 2. Вызываются release на всех strong свойствах
    // 3. Можно вручную вызвать [super dealloc];
    // 4. После этого объект считается мёртвым
}
```

**В Swift (ARC):**

- `deinit` — это **Swift-обёртка** над `objc dealloc`
- Вызывается **после** уменьшения retain count до 0
- **Порядок выполнения в deinit:**
  1. Выполняется тело `deinit` (твой код)
  2. Освобождаются свойства (ARC делает release)
  3. Вызывается `[super dealloc]` (если класс наследует NSObject)
  4. Объект помечается как deallocated — дальнейшие вызовы приводят к crash (EXC_BAD_ACCESS)

**Важные факты senior-уровня:**

- `deinit` **не вызывается** на [[struct]] и [[enum]] — они [[value type]]s, не имеют [[retain count]]
- `deinit` вызывается **только один раз** за жизнь объекта
- Внутри `deinit` **нельзя** вызывать методы, которые могут привести к retain (например, `self.someClosure = nil` может быть опасно)
- `deinit` — **последнее место**, где можно безопасно освободить ресурсы (invalidate таймеры, remove observers)
- **Swift 6** (2025–2026) усиливает правила: строгий контроль за `weak`/`unowned`, предупреждения о потенциальных циклах

**Схема жизненного цикла (текстовая)**

```
alloc → init → retain count = 1
   ↓
использование → retain / release (ARC)
   ↓
retain count == 0
   ↓
deinit вызывается
   ↓
твой код в deinit
   ↓
освобождение свойств (ARC release)
   ↓
[super dealloc] (если NSObject)
   ↓
объект уничтожен — память возвращена в кучу
```

**Типичные места, где deinit критичен в 2026:**

- Таймеры (`Timer`, `CADisplayLink`)
- KVO (`observe`, `removeObserver`)
- NotificationCenter observers
- [[URLSessionTask]] / [[URLSessionDataTask]] (cancel)
- [[AVPlayer]] / [[AVAudioPlayer]] (pause, remove observers)
- Closure-based callbacks (location manager, network callbacks)
- Combine cancellables (AnyCancellable)

**Рекомендация senior-уровня 2026:**

```swift
class SafeTimerController {
    private var timer: Timer?
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            self?.tick()
        }
        
        // Combine пример
        somePublisher
            .sink { [weak self] value in self?.handle(value) }
            .store(in: &cancellables)
    }
    
    deinit {
        timer?.invalidate()
        timer = nil
        // cancellables автоматически отменяются при deinit
        print("SafeTimerController уничтожен")
    }
    
    private func tick() { /* ... */ }
}
```

**Короткий итог 2026:**

> `dealloc` / `deinit` — **последний шанс** объекта освободить ресурсы перед смертью.  
> Junior: если `deinit` не вызывается → утечка памяти (retain cycle).  
> Middle: используй `weak self`, `unowned`, `invalidate`, `removeObserver`.  
> Senior: `deinit` — индикатор здоровья архитектуры; измеряй утечки через Instruments → Leaks / Allocations; Swift 6 делает retain cycle более явными.  

Если хочешь — могу показать:
- полный чек-лист самых частых retain cycle в 2026 году
- как использовать `deinit` для отладки (print + OSLog)
- как Instruments помогает находить утечки, связанные с dealloc/deinit

Удачи с чистой памятью и нулевыми утечками в твоём проекте! 🧹