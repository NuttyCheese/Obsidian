**`inout`** — это модификатор параметра функции, который позволяет **передавать аргумент по ссылке** ([[reference semantic]]), а не по значению ([[value semantic]]).

Это даёт функции возможность **изменять оригинальную переменную**, переданную извне.

### 1. Почему и когда нужен inout

| Тип передачи | Семантика                  | Изменение внутри функции видно снаружи? | Пример |
|--------------|----------------------------|-------------------------------------------|--------|
| Обычный параметр | По значению (копируется)   | Нет                                       | `func increment(_ n: Int)` |
| `inout` параметр | По ссылке (не копируется)  | Да                                        | `func increment(_ n: inout Int)` |
| `inout` + `&`    | Явная передача ссылки      | Да                                        | `increment(&myVar)` |

**Главное правило**:
> `inout` нужен, когда функция должна **изменить переменную вызывающей стороны**, а возвращать новое значение неудобно или невозможно.

### 2. Синтаксис и базовые правила

- Параметр объявляется как `inout Тип`
- При вызове перед переменной ставится `&` (амперсанд)
- `inout` работает **только** с **[[var]]**-переменными (не с [[let]])
- `inout` нельзя использовать с константами, литералами, результатами вычислений

```swift
func double(_ n: inout Int) {
    n *= 2
}

var number = 10
double(&number)     // OK → number = 20

// let number = 10
// double(&number)  // Ошибка: Cannot pass immutable value as inout argument

// double(&42)      // Ошибка: Cannot pass immutable value
// double(&(5 + 5)) // Ошибка: Cannot pass immutable value
```

### 3. Полные примеры кода (от простого к продвинутому)

#### Пример 1 — Простое изменение переменной

```swift
func increment(by amount: Int = 1, _ value: inout Int) {
    value += amount
}

var score = 0
increment(&score)           // score = 1
increment(by: 5, &score)    // score = 6
print(score)                // 6
```

#### Пример 2 — Обмен значениями (классика)

```swift
func swap<T>(_ a: inout T, _ b: inout T) {
    (a, b) = (b, a)
}

var x = 10
var y = 20
swap(&x, &y)
print(x, y) // 20 10
```

#### Пример 3 — Работа с массивом внутри структуры

```swift
struct ShoppingCart {
    private var items: [String] = []
    
    mutating func add(_ item: String) {
        items.append(item)
    }
    
    func totalItems() -> Int {
        items.count
    }
}

var cart = ShoppingCart()
cart.add("iPhone")
cart.add("AirPods")
print(cart.totalItems()) // 2
```

**Здесь `mutating` заменяет `inout` на уровне метода структуры.**

#### Пример 4 — inout в [[generic]]-функции

```swift
func sortAscending<T: Comparable>(_ a: inout T, _ b: inout T) {
    if a > b {
        swap(&a, &b)
    }
}

var price1 = 150.0
var price2 = 99.99
sortAscending(&price1, &price2)
print(price1, price2) // 99.99 150.0
```

#### Пример 5 — inout с несколькими параметрами + возвращаемое значение

```swift
func divideAndRemainder(_ dividend: Int, by divisor: Int, quotient: inout Int, remainder: inout Int) {
    quotient = dividend / divisor
    remainder = dividend % divisor
}

var q = 0
var r = 0
divideAndRemainder(17, by: 5, quotient: &q, remainder: &r)
print("17 ÷ 5 = \(q), остаток \(r)") // 17 ÷ 5 = 3, остаток 2
```

#### Пример 6 — inout в замыкании (capture by reference)

```swift
func modifyLater(_ value: inout Int) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
        value += 100
        print("Значение изменено асинхронно:", value)
    }
}

var counter = 0
modifyLater(&counter)
print("Сразу после вызова:", counter) // 0

// Через 2 секунды увидим:
// Значение изменено асинхронно: 100
```

**Важно**: `inout` сохраняет ссылку даже при асинхронной работе.

### 4. Таблица: inout vs обычные параметры vs возвращаемое значение

| Способ изменения  | Изменение видно снаружи? | Кол-во возвращаемых значений | Читаемость | Производительность    | Пример                                                      |
| ----------------- | ------------------------ | ---------------------------- | ---------- | --------------------- | ----------------------------------------------------------- |
| Обычный параметр  | Нет                      | 1 (возврат)                  | Высокая    | Высокая               | `func doubled(_ n: Int) -> Int`                             |
| inout             | Да                       | 0–много                      | Средняя    | Высокая               | `func increment(_ n: inout Int)`                            |
| Возврат [[tuple]] | Да                       | Несколько                    | Высокая    | Средняя (копирование) | `func divmod(_ a: Int, _ b: Int) -> (q: Int, r: Int)`       |
| Возврат структуры | Да                       | 1 (структура)                | Высокая    | Средняя               | `func movedPoint(from p: Point, dx: Int, dy: Int) -> Point` |

### 5. Реальные сценарии в iOS-разработке (2026)

#### Сценарий 1 — Обновление состояния в ViewModel (без inout)

```swift
@MainActor
class CounterViewModel: ObservableObject {
    @Published var count = 0
    
    func increment() {
        count += 1  // нет inout — @Published меняется через reference
    }
}
```

#### Сценарий 2 — Работа с массивом в структуре ([[mutating method|mutating]] + inout)

```swift
struct TodoList {
    private var todos: [String] = []
    
    mutating func add(_ todo: String) {
        todos.append(todo)
    }
    
    mutating func remove(at index: Int) {
        guard index < todos.count else { return }
        todos.remove(at: index)
    }
}

var list = TodoList()
list.add("Купить молоко")
list.add("Позвонить маме")
list.remove(at: 0)
```

#### Сценарий 3 — inout в алгоритмах (сортировка пузырьком)

```swift
func bubbleSort<T: Comparable>(_ array: inout [T]) {
    var swapped = true
    while swapped {
        swapped = false
        for i in 0..<(array.count - 1) {
            if array[i] > array[i + 1] {
                array.swapAt(i, i + 1)
                swapped = true
            }
        }
    }
}

var numbers = [64, 34, 25, 12, 22, 11, 90]
bubbleSort(&numbers)
print(numbers) // [11, 12, 22, 25, 34, 64, 90]
```

### 6. Типичные ошибки и ловушки

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Вызов inout на let                          | Ошибка компиляции                            | Использовать `var` |
| Передача литерала или выражения             | Ошибка компиляции                            | Передавать только var-переменную |
| inout в замыкании без захвата              | Ошибка или неожиданное поведение             | Использовать `[inout]` или захват явно |
| Забыть & при вызове                         | Ошибка компиляции                            | Всегда ставить `&` перед переменной |
| Использование inout вместо возврата         | Код становится менее читаемым                | Для 1–2 значений лучше возвращать tuple |

### 7. Лучшие практики 2026 года

- Используй `inout` **только** когда нужно изменить **несколько** переменных
- Для одного значения — предпочитай **возврат** нового значения (функциональный стиль)
- Для коллекций внутри структур — используй **mutating** методы (push, pop, append)
- В многопоточных сценариях — `inout` **не безопасен** без [[actor]] или [[@MainActor]]
- Для immutable данных — возвращай копию с изменениями
- В [[SwiftUI]] — `@State` и `@Binding` заменяют inout в большинстве случаев

**Короткий девиз**:
> «`inout` — это когда функция должна **изменить** твою переменную, а не просто её прочитать или вернуть новое значение.  
> Используй с осторожностью — лучше возвращать новое значение, чем менять по ссылке.»
