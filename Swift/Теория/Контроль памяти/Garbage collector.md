#memory_control #Swift 
# Управление памятью в Swift: ARC и циклы ссылок

[[Swift]] **не использует сборщик мусора** (Garbage Collector), как Java, C# или Go.  
Вместо этого применяется **Automatic Reference Counting ([[ARC]])** — полностью **детерминированный** механизм, работающий на этапе компиляции.

### Как работает ARC

1. Каждый объект класса ([[class]]) имеет **счётчик ссылок** ([[retain count]]).
2. При создании **сильной ссылки** ([[strong]]) → retain count +1
3. При уничтожении сильной ссылки ([[nil]], выход из scope) → retain count -1
4. Когда retain count становится **0** → вызывается [[deinit]] и память освобождается **немедленно**.

```swift
class User {
    let name: String
    deinit { print("\(name) уничтожен") }
}

var user: User? = User(name: "Анна")  // retain count = 1
var another = user                    // retain count = 2
user = nil                            // retain count = 1
another = nil                         // retain count = 0 → deinit → "Анна уничтожен"
```

### Три вида ссылок

| Вид ссылки      | Синтаксис                        | Увеличивает retain count? | Зануляется при dealloc? | Когда использовать                              |
| --------------- | -------------------------------- | ------------------------- | ----------------------- | ----------------------------------------------- |
| **[[strong]]**  | [[var]] / [[let]] (по умолчанию) | Да                        | Нет                     | Обычные свойства, владение объектом             |
| **[[weak]]**    | `weak var`                       | Нет                       | Да (становится [[nil]]) | Разрыв цикла, делегаты, datasource              |
| **[[unowned]]** | `unowned let/var`                | Нет                       | Нет                     | Когда объект гарантированно живёт дольше ссылки |

### Классический [[retain cycle]] (утечка памяти)

```swift
class Parent {
    var child: Child?
}

class Child {
    var parent: Parent?          // ← сильная ссылка → цикл!
}

let p = Parent()
let c = Child()
p.child = c
c.parent = p
// p и c никогда не будут освобождены → утечка
```

### Как разрывать цикл

#### Вариант 1 — weak (самый безопасный)

```swift
class Child {
    weak var parent: Parent?     // retain count не увеличивается
}
```

#### Вариант 2 — unowned (быстрее, но опаснее)

```swift
class Child {
    unowned let parent: Parent   // не Optional, краш при доступе к мёртвому объекту
}
```

**Правило 2026 года**:
> Используй `weak` по умолчанию.  
> `unowned` — только если 100% уверен, что объект-владелец живёт дольше (parent → child, [[lazy]] var после [[init]]).

### Самая частая утечка — замыкания

```swift
class TimerController {
    var timer: Timer?
    
    func start() {
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            self.tick()          // сильный захват self → цикл!
        }
    }
    
    func tick() { print("Тик") }
}
```

**Исправление**:

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

или (Swift 5.3+):

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] in
    self?.tick()
}
```

### Ключевые правила ARC в 2026 году

- [[ARC]] работает **только** с class ([[reference type]])
- [[struct]], [[enum]], [[tuple]], [[Array]], [[Dictionary]], [[String]] — [[value type]] → **не попадают под ARC**
- Коллекции используют **[[Copy-on-Write]]** (COW) — копирование происходит только при мутации
- `deinit` вызывается **детерминированно** и **сразу** после RC = 0
- Циклы сильных ссылок — **единственная причина утечек памяти** в ARC
- Главные источники циклов:  
  1. Замыкания, захватывающие [[self]]  
  2. Делегаты / datasource / parent-child отношения  
  3. Два объекта, сильно ссылающиеся друг на друга

**Золотое правило**:
> «Внутри класса, в замыканиях и при ссылках на владельца — пиши `[weak self]` или `weak var`.  
> `unowned` — только когда ты **точно** знаешь, что объект не умрёт раньше.»
