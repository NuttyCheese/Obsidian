**Compile-time** и **[[Runtime]]** — два главных этапа жизненного цикла любого [[Swift]]-приложения.  
Понимание разницы между ними — один из самых важных навыков хорошего [[iOS]]-разработчика.

### Коротко и чётко (таблица 2026)

| Характеристика         | Compile-time (время компиляции)                                    | Runtime (время выполнения)                                 |
| ---------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------- |
| Когда происходит       | Пока ты жмёшь **Build** / **Cmd+B** / **Cmd+R** (до запуска)       | Когда приложение **запущено** и реально работает           |
| Кто работает           | Компилятор Swift + LLVM                                            | Процессор + Swift runtime + ОС ([[iOS]], macOS)            |
| Что проверяет / делает | Типы, синтаксис, протоколы, диапазоны, generics, макросы           | Всё остальное: создание объектов, вызовы, сеть, UI, память |
| Ошибки                 | **Compile error** — приложение **не соберётся**                    | **Crash** / **exception** / **undefined behavior**         |
| Стоимость ошибок       | Бесплатно (не доходишь до пользователя)                            | Дорого (пользователь видит краш, App Store рейтинг падает) |
| Примеры ошибок         | `let x: Int = "hello"`<br>`Int8(1000)`<br>`let arr: [Int] = ["a"]` | `nil!`<br>`index out of range`<br>`force unwrap nil`       |
| Примеры проверок       | Type inference, ExpressibleByIntegerLiteral, where-констрейнты     | `if let`, `guard let`, `as?`, `try?`, `fatalError`         |
| Можно ли выполнить код | Почти нет (только макросы и compile-time константы)                | Да — весь основной код                                     |
| Оптимизация            | Максимальная (inlining, dead code elimination, devirtualization)   | Минимальная ([[ARC]], динамическая диспетчеризация)        |

### Самые важные примеры (что ловится на compile-time vs runtime)

#### Compile-time ошибки (приложение даже не соберётся)

```swift
// 1. Несовместимые типы
let x: Int = "hello"                    // Cannot convert value of type 'String' to 'Int'

// 2. Выход за диапазон литерала
let a: Int8 = 200                       // Integer literal '200' overflows when stored into 'Int8'

// 3. Нарушение протокола
struct User: Equatable {                // OK
    let id: Int
}
let users: [User] = [User(id: 1)]
users.contains(User(id: 1))             // OK (Equatable)

// Но если забыть Equatable:
struct UserNoEq { let id: Int }
let usersNoEq: [UserNoEq] = []          
usersNoEq.contains(UserNoEq(id: 1))     // No exact matches in call to instance method 'contains'

// 4. Generic constraint violation
func sum<T: Numeric>(_ values: [T]) -> T { ... }
sum([1, "2"])                           // 'String' does not conform to 'Numeric'
```

#### Runtime ошибки (приложение компилируется, но падает при запуске)

```swift
// 1. Force unwrap nil
let name: String! = nil
print(name.count)                       // Fatal error: Unexpectedly found nil while unwrapping an Optional value

// 2. Index out of range
let arr = [1, 2, 3]
print(arr[10])                          // Fatal error: Index out of range

// 3. Downcast failure с as!
let view: UIView = UILabel()
let button = view as! UIButton          // Fatal error: Could not cast value of type 'UILabel' to 'UIButton'

// 4. Деление на ноль
let x = 10 / 0                          // Fatal error: Division by zero
```

### 3. Почему Swift так сильно давит на compile-time (философия 2026)

Swift — это язык, который **сознательно переносит максимум проверок на compile-time**, чтобы минимизировать runtime-краши.

Главные принципы:

- **Type safety** — ошибки типов ловятся сразу  
- **No implicit conversions** — нет неявных приведений (как в Objective-C)  
- **Exhaustive checks** — `switch` должен покрывать все case  
- **Literal protocols** — `ExpressibleByIntegerLiteral`, `ExpressibleByStringLiteral` и т.д. проверяют диапазоны на этапе компиляции  
- **Macros и compile-time evaluation** — Swift 5.9+ позволяет выполнять код на этапе компиляции (#file, #line, макросы)  
- **Strict concurrency** — в Swift 6 почти все нарушения конкурентности ловятся на compile-time

Результат:  
Хороший Swift-код → **очень мало runtime-крашей** → высокая надёжность → меньше баг-репортов → лучше рейтинг в App Store.

### 4. Полезные вопросы для самопроверки (чтобы закрепить)

1. Что произойдёт раньше: проверка `Int8(1000)` или `nil!`?  
   → `Int8(1000)` — compile-time ошибка, приложение не соберётся.

2. Где ловится ошибка `index out of range`?  
   → Runtime (в момент обращения к массиву).

3. Можно ли на compile-time проверить, что массив не пустой?  
   → Нет, `!array.isEmpty` — это runtime-проверка.

4. Почему `let x: Int = "42"` — ошибка компиляции, а `Int("42")!` — может крашнуться на runtime?  
   → Первое — несоответствие типов (compile-time), второе — неудачный парсинг (runtime).

### 5. Короткий девиз 2026

> **Compile-time** — это когда компилятор говорит: «я уже всё проверил, иди дальше».  
> **Runtime** — это когда программа говорит: «ой, кажется я упала».  
> Цель хорошего Swift-разработчика — **максимум ошибок на compile-time**, **минимум на runtime**.
