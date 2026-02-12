#memory_management #reference_counting #arc #ios #swift #objective_c #retain_release #object_lifecycle #heap #automatic_reference_counting
**Счётчик ссылок** (Reference Counting) — это механизм, который отслеживает, сколько **сильных ссылок** (strong references) указывает на объект. Когда счётчик достигает **0**, объект автоматически освобождается из памяти.

[[Swift]] использует **автоматический счётчик ссылок** — **[[ARC]]** (Automatic Reference Counting).  
ARC полностью **детерминирован**, работает на этапе компиляции и **не имеет** сборщика мусора (GC).

### Как работает счётчик ссылок (пошагово)

1. Создаётся объект → **[[retain count]] = 1**
2. Создаётся новая сильная ссылка → **retain count +1**
3. Сильная ссылка уничтожается ([[nil]], выход из scope) → **retain count -1**
4. retain count = 0 → вызывается [[deinit]] → память освобождается **немедленно**

```swift
class User {
    let name: String
    deinit { print("\(name) уничтожен") }
}

var user: User? = User(name: "Анна")   // RC = 1
var copy = user                        // RC = 2
user = nil                             // RC = 1
copy = nil                             // RC = 0 → deinit → "Анна уничтожен"
```

### Три вида ссылок в ARC

| Вид ссылки  | Синтаксис                        | Влияет на retain count? | Зануляется при dealloc? | Когда использовать                              |
| ----------- | -------------------------------- | ----------------------- | ----------------------- | ----------------------------------------------- |
| **strong**  | [[var]] / [[let]] (по умолчанию) | Да                      | Нет                     | Обычные свойства, владение объектом             |
| **weak**    | [[weak]] `var`                   | Нет                     | Да (становится `nil`)   | Разрыв цикла, делегаты, datasource              |
| **unowned** | [[unowned]] `let/var`            | Нет                     | Нет                     | Когда объект гарантированно живёт дольше ссылки |

### Самая частая причина утечек — цикл сильных ссылок

```swift
class Parent {
    var child: Child?
}

class Child {
    var parent: Parent?          // ← сильная ссылка → цикл
}

let p = Parent()
let c = Child()
p.child = c
c.parent = p
// p и c никогда не освободятся → утечка памяти
```

### Как избежать цикла

1. Делай одну из ссылок **weak** или **unowned**

```swift
class Child {
    weak var parent: Parent?     // безопасно
    // или
    unowned let parent: Parent   // быстрее, но опасно
}
```

2. В замыканиях всегда используй `[weak self]` (по умолчанию)

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

### Ключевые факты ARC (2026)

- Работает **только** с class (reference type)
- [[struct]], [[enum]], [[Array]], [[Dictionary]], [[String]] — [[value type]] → **не используют [[ARC]]** ([[Copy-on-Write]])
- `deinit` вызывается **детерминированно** и **сразу** после RC = 0
- **Нет GC** — нет пауз, нет поиска живых объектов
- ARC **не разрывает** циклы автоматически — это ответственность разработчика
- Самые частые утечки:
  1. Замыкания без `[weak self]`
  2. Делегаты / datasource / parent-child без `weak`
  3. Таймеры / наблюдатели без отписки

### Короткий чек-лист (чтобы не было утечек)

- В замыканиях внутри классов → **всегда** `[weak self]`
- Делегаты, datasource, observers → **всегда** `weak`
- Таймеры, CADisplayLink, Notification → invalidate/remove в `deinit`
- `deinit` с логом → проверяй, вызывается ли он
- Используй **Memory Graph Debugger** и **Instruments → Leaks** после навигации по экранам

**Золотое правило Swift**:
> «Внутри класса, в замыканиях и при ссылках на владельца — пиши `[weak self]` или `weak var`.  
> `unowned` — только если ты **точно** знаешь, что объект не умрёт раньше.»
