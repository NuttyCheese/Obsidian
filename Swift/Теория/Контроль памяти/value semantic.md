**Value semantics** (семантика значений) в [[Swift]] — это ключевая особенность языка, которая отличает его от большинства других современных языков программирования.

### Что такое value semantics простыми словами

Когда ты присваиваешь, передаёшь или копируешь значение переменной — создаётся **независимая копия** данных.  
Изменение одной копии **никогда** не влияет на другие копии.

Это противоположность **reference semantics** (семантика ссылок), где присваивание копирует только **ссылку** (указатель), а само значение остаётся общим.

### Основные типы со value semantics в Swift (2026)

| Тип / Категория                         | Примеры                                                        | Копируется ли при присваивании? | Изменение одной копии влияет на другие? | Sendable по умолчанию?          |
| --------------------------------------- | -------------------------------------------------------------- | ------------------------------- | --------------------------------------- | ------------------------------- |
| **Все структуры**                       | [[struct]], [[enum]], [[tuple]]                                | **Да**                          | **Нет**                                 | Да (если свойства [[Sendable]]) |
| **Основные типы Foundation**            | [[String]], [[Data]], [[Date]], [[UUID]], [[URL]], [[Decimal]] | **Да**                          | **Нет**                                 | Да                              |
| **Коллекции**                           | [[Array]]<T>, [[Dictionary]]<Key, Value>, [[Set Collection]]<T>           | **Да** (Copy-on-Write)          | **Нет**                                 | Да (если T/Key/Value Sendable)  |
| **Числовые типы**                       | [[Int]], [[UInt]], [[Float]], [[Double]], [[Bool]], Float16    | **Да**                          | **Нет**                                 | Да                              |
| **[[CGPoint]], [[CGRect]], [[CGSize]]** | Core Graphics структуры                                        | **Да**                          | **Нет**                                 | Да                              |

### Примеры value semantic vs [[reference semantic]]

```swift
// Value semantics — struct
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 10, y: 20)
var p2 = p1           // создаётся полная копия
p2.x = 50

print(p1.x)           // 10 — не изменился
print(p2.x)           // 50

// Reference semantics — class
class Person {
    var name = "Alice"
}

let personA = Person()
let personB = personA   // копируется только ссылка
personB.name = "Bob"

print(personA.name)     // "Bob" — изменилось для всех
print(personB.name)     // "Bob"
```

### Самые важные особенности value semantics в Swift 2026

1. **[[Copy-On-Write]] (CoW)**  
   Большинство коллекций (Array, Dictionary, Set, String, Data) используют **ленивое копирование**:  
   - Пока ты не мутируешь копию — она делит память с оригиналом (дешёво)  
   - Как только мутируешь — создаётся независимая копия

```swift
var array1 = [1, 2, 3]          // выделена память
var array2 = array1             // array2 указывает на ту же память (CoW)
array2.append(4)                // здесь создаётся новая копия массива
// array1 остаётся [1,2,3]
```

2. **Value semantics = иммутабельность по умолчанию**  
   Если все свойства `let` → экземпляр полностью неизменяемый:

```swift
struct ImmutableUser {
    let id: UUID
    let name: String
}

let user = ImmutableUser(id: UUID(), name: "Alice")
// user.name = "Bob" // Ошибка компиляции
```

3. **Value semantics + Swift Concurrency**  
   Почти все типы со value semantics **автоматически Sendable** — их можно безопасно передавать между задачами и актёрами.

```swift
actor Cache {
    var users: [String: User] = [:]  // [String: User] — Sendable, если User — struct
}
```

### Когда value semantics ломается (ловушки)

1. **[[struct]] содержит ссылочный тип** ([[class]], [[actor]])

```swift
struct Container {
    var user: PersonClass   // PersonClass — reference semantics
}

var c1 = Container(user: PersonClass())
var c2 = c1
c2.user.name = "Bob"     // изменит и c1.user.name
```

2. **NSMutable* коллекции** (NSMutableArray, NSMutableDictionary) — reference semantics

3. **Existential types** ([[any Protocol]]) — боксинг → reference semantics

### Лучшие практики value semantics в Swift 2026

- **Предпочитай struct / enum** для большинства типов данных  
- **Делай свойства let по умолчанию** — иммутабельность = безопасность  
- **Избегай class** — если только не нужна именно reference semantics (делегаты, view controllers, shared state)  
- **actor** — для mutable shared state вместо class + lock  
- **final class + immutable свойства** — если нужен reference type (экономия vtable)  
- **@MainActor** — для UI-классов ([[UIViewController]], ViewModel)  
- **Sendable** — обязательно для всех типов, передаваемых между задачами  
- **Документируйте** — пиши комментарий «value semantics — копия независима»

**Короткий девиз 2026**:
> «Value semantics — это когда каждая копия действительно независима.  
> В Swift 2026 это **основа** безопасного, предсказуемого и конкурентного кода.  
> Главное правило: struct / enum / String / Array — почти всегда value semantics.  
> class / actor — reference semantics (опасно без @MainActor / Sendable).»
