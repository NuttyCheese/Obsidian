**`any`** в [[Swift]] — это **явный маркер [[existential type]]** (экзистенциального типа), который говорит компилятору:  
«Здесь может быть **любой** тип, который соответствует этому протоколу, но я не знаю (и мне не важно) какой именно».

С 2022–2023 годов (Swift 5.7 и особенно Swift 6) использование `any` стало **обязательным** в большинстве случаев, когда раньше протокол использовался как тип напрямую. Это одно из самых важных изменений в языке за последние годы.

### 1. Почему ввели обязательный `any` (короткая история)

До Swift 5.7 протоколы можно было использовать как типы **неявно**:

```swift
protocol Drawable {
    func draw()
}

let shape: Drawable = Circle()  // OK в Swift 5.6 и раньше
```

Это создавало **неявный existential type** — коробку, в которой может лежать любой Drawable.

Но у такого подхода были проблемы:
- **Производительность** — всегда [[dynamic dispatch]] (медленнее)
- **Неоднозначность** — компилятор не знал, можно ли хранить несколько разных типов в массиве
- **Сложности с [[generic]]** и **[[associatedtype]]s**
- **Проблемы со строгой конкурентностью** (Swift 6)

С Swift 5.7+ ввели правило:

> Если вы хотите использовать протокол как **тип** (в переменной, параметре, возвращаемом значении, массиве) — **явно пишите `any`**.

Без `any` теперь ошибка:

```swift
let shape: Drawable = Circle()  
// Ошибка: Protocol 'Drawable' can only be used as a generic constraint
// because it has Self or associated type requirements
```

Правильно:

```swift
let shape: any Drawable = Circle()  // OK
```

### 2. Когда писать `any`, а когда нет (таблица 2026)

| Ситуация                               | Писать `any`? | Пример правильного кода                             | Почему                         |
| -------------------------------------- | ------------- | --------------------------------------------------- | ------------------------------ |
| Переменная / константа хранит значение | **Да**        | `let shape: any Drawable = Circle()`                | Existential type               |
| Параметр функции / возвращаемый тип    | **Да**        | `func draw(_ shape: any Drawable)`                  | Existential                    |
| Массив / коллекция разных реализаций   | **Да**        | `[any Drawable]`                                    | Разные типы                    |
| Generic constraint (обобщённый тип)    | **Нет**       | `func draw<T: Drawable>(_ shape: T)`                | Concrete type, static dispatch |
| Associated type в протоколе            | **Нет**       | `associatedtype Shape: Drawable`                    | Constraint, не тип             |
| [[Self]] в протоколе                   | **Нет**       | `associatedtype Element: Equatable`                 | Constraint                     |
| Протокол без Self / associatedtype     | **Да**        | `protocol Loggable { func log() }` → `any Loggable` | Existential                    |

### 3. Самые частые и важные примеры 2026 года

#### Пример 1. Массив разных реализаций (самый популярный кейс)

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable { func draw() { print("○") } }
struct Square: Drawable { func draw() { print("□") } }

let shapes: [any Drawable] = [Circle(), Square(), Circle()]

for shape in shapes {
    shape.draw()  // ○ □ ○
}
```

Без `any` — ошибка компиляции.

#### Пример 2. Функция с existential возвращаемым типом

```swift
func makeRandomShape() -> any Drawable {
    Bool.random() ? Circle() : Square()
}

let random = makeRandomShape()
random.draw()
```

#### Пример 3. Generic vs Existential (разница в производительности)

```swift
// Existential — dynamic dispatch (медленнее)
func drawExistential(_ shape: any Drawable) {
    shape.draw()
}

// Generic — static dispatch (быстрее)
func drawGeneric<T: Drawable>(_ shape: T) {
    shape.draw()
}
```

В 2026 году **рекомендуется** использовать generic, когда тип известен на этапе компиляции.

#### Пример 4. `any` + [[async]] / [[Sendable]] (Swift 6)

```swift
protocol AsyncTask: Sendable {
    func execute() async
}

func runTasks(_ tasks: [any AsyncTask]) async {
    await withTaskGroup(of: Void.self) { group in
        for task in tasks {
            group.addTask {
                await task.execute()
            }
        }
    }
}
```

Здесь `any AsyncTask` работает, потому что протокол помечен [[Sendable]].

### 4. Лучшие практики использования `any` в Swift 2026

- **Пишите `any` явно** — это требование Swift 6 strict mode  
- **Предпочитайте generic** (`<T: Protocol>`) там, где тип известен — быстрее и безопаснее  
- **Используйте `any` только когда нужно хранить разные типы** (массивы, коллекции, возвращаемые значения)  
- **Не злоупотребляйте existential** — они медленнее и ограничивают возможности (нельзя использовать associated types без type erasure)  
- **Для протоколов с `Self` / `associatedtype`** — `any` запрещён, используйте generic или type erasure (AnyDrawable)  
- **Документируйте** — пиши комментарий «[any Drawable] — массив разных фигур для рендеринга»

**Короткий девиз 2026**:
> `any Protocol` — это **коробка**, в которой может лежать **любой** тип, соответствующий протоколу.  
> В Swift 6+ писать `any` **обязательно**, когда протокол используется как тип.  
> Если тип известен — используйте generic `<T: Protocol>`.  
> `any` — для гибкости, generic — для скорости и безопасности.
