**Reference semantics** (семантика ссылок) в Swift — это способ, при котором **значение** передаётся **по ссылке** (reference), а не по значению (value). Это противоположность **value semantics** (семантика значений), которая является основой большинства Swift-типов.

В 2026 году понимание reference semantics критически важно, особенно в контексте **Swift 6 strict concurrency**, **Sendable**, **actor** и **memory management**.

### Основные типы по семантике в Swift

| Семантика               | Типы / Классы, которые её используют | Копирование при присваивании | Mutability | Sendable? | Пример |
|--------------------------|---------------------------------------|-------------------------------|------------|-----------|--------|
| **Value semantics**      | struct, enum, tuple, Array, Dictionary, Set, String, Data, Date, CGPoint и т.д. | Копируется полностью (CoW — copy-on-write) | Изменение одной копии не влияет на другие | Да (если все свойства Sendable) | `var a = [1,2]; var b = a; b.append(3); // a остаётся [1,2]` |
| **Reference semantics**  | class, Actor, AnyObject-типы, NSObject-подклассы | Копируется **ссылка** (указатель) | Изменение влияет на все ссылки | Нет (если не помечен Sendable) | `class User { var name = "" }; let a = User(); let b = a; b.name = "Bob"; // a.name тоже "Bob"` |
| **Existential container** | `any Protocol` (экзистенциальный тип) | Копируется контейнер + боксинг | Зависит от типа внутри | Частично (Sendable, если протокол Sendable) | `let views: [any View]` — разные типы, но все боксятся |

### Почему reference semantics опасны в Swift 6+

1. **Data races** — если несколько задач одновременно мутируют объект по ссылке → undefined behavior  
   → actor и Sendable решают эту проблему

2. **Retain cycles** — сильные ссылки между объектами → утечки памяти  
   → weak / unowned / [weak self]

3. **Непредсказуемость** — изменение одной переменной влияет на все, кто держит ссылку

4. **Трудности с тестированием** — mutable shared state сложно мокать и изолировать

### Как избежать проблем reference semantics в 2026

| Проблема                              | Решение в Swift 6+                                   | Пример |
|---------------------------------------|------------------------------------------------------|--------|
| Data race при мутации                 | actor / @MainActor / Sendable                        | `actor UserCache { var users: [User] = [] }` |
| Retain cycle                          | weak / unowned / [weak self]                         | `weak var delegate: Delegate?` |
| Shared mutable state                  | Prefer value types (struct) + immutable по умолчанию | `struct User { let name: String }` |
| Existential overhead (any Protocol)   | Prefer `some Protocol` или generics                  | `func makeView() -> some View` |
| Unsafe reference (class)              | Mark as `final` / `Sendable` / `@unchecked Sendable` | `final class Logger: @unchecked Sendable { ... }` |

### Пример: reference vs value semantics

```swift
class UserClass {  // reference semantics
    var name = "Alice"
}

struct UserStruct {  // value semantics
    var name = "Alice"
}

var classA = UserClass()
var classB = classA
classB.name = "Bob"
print(classA.name)  // "Bob" — одна и та же ссылка

var structA = UserStruct()
var structB = structA
structB.name = "Bob"
print(structA.name) // "Alice" — копия
```

### Лучшие практики reference semantics в Swift 2026

- **Предпочитай value semantics** (struct, enum) для большинства типов данных  
- **class** — только когда нужна reference semantics (делегаты, view controllers, сервисы, shared state)  
- **actor** — для mutable shared state в concurrency  
- **final class** — если класс не предназначен для наследования (экономия vtable)  
- **Sendable** — обязательно для всех классов, передаваемых между задачами  
- **@MainActor** — для всего UI-related (UIViewController, ViewModel)  
- **Документируйте** — пиши комментарий «reference semantics — shared mutable state»

**Короткий девиз 2026**:
> «Reference semantics — это когда изменение одной переменной меняет все копии, потому что они указывают на один и тот же объект в памяти.  
> В 2026 году это **опасно** без actor / Sendable / weak.  
> Основное правило: используй class только когда тебе **действительно** нужна общая ссылка, во всём остальном — struct и value semantics.»

Удачи с безопасным и предсказуемым кодом в Swift! 🔗