## 1. Что такое `enum`

**`enum` (enumeration)** — это **тип данных, который определяет набор связанных значений (cases)**.

- Используется для моделирования ограниченного набора вариантов
    
- Может содержать **свойства**, **методы**, **ассоциированные значения**, **raw values**
    
- Позволяет писать **чистый и безопасный код**, избегая магических чисел или строк
    

> Проще говоря: enum = «тип, который может быть только одним из заранее определённых вариантов».

---

## 2. Основные термины

| Термин                   | Описание                                                              |
| ------------------------ | --------------------------------------------------------------------- |
| **Case**                 | Конкретный вариант enum                                               |
| **Raw value**            | Значение, которое напрямую хранится ([[String]], [[Int]] и т.д.)      |
| **[[Associated value]]** | Данные, связанные с конкретным case                                   |
| **Methods**              | Функции внутри enum                                                   |
| **Computed properties**  | Свойства внутри enum, вычисляемые динамически                         |
| **Mutating**             | Ключевое слово для изменения self внутри enum, если он [[Value Type]] |

---

## 3. Основной синтаксис

```swift
enum Direction {
    case north
    case south
    case east
    case west
}

let dir: Direction = .north
```

- Enum определяется через `case`
    
- Созданная переменная типа enum может принимать **только указанные значения**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший enum

```swift
enum CompassPoint {
    case north, south, east, west
}

let direction = CompassPoint.east
print(direction) // east
```

- Несколько case в одну строку
    

---

### Пример 2. Enum с raw value

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars
}

let earth = Planet.earth
print(earth.rawValue) // 3
```

- Raw value = автоматически присваиваемое число или строка
    
- Можно инициализировать через `Planet(rawValue: 2)`
    

---

### Пример 3. Enum с associated value

```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}

let productBarcode = Barcode.upc(8, 85909, 51226, 3)
let qr = Barcode.qrCode("ABCDEFG")
```

- Позволяет хранить **данные внутри case**
    
- Используется для более сложных моделей
    

---

### Пример 4. Enum с методами

```swift
enum Direction {
    case north, south, east, west
    
    func description() -> String {
        switch self {
        case .north: return "Up"
        case .south: return "Down"
        case .east: return "Right"
        case .west: return "Left"
        }
    }
}

print(Direction.north.description()) // Up
```

- Enum может содержать **методы и computed properties**
    

---

### Пример 5. Enum + [[mutating method]]

```swift
enum SwitchState {
    case on, off
    
    mutating func toggle() {
        self = (self == .on) ? .off : .on
    }
}

var light = SwitchState.off
light.toggle()
print(light) // on
```

- `mutating` нужен, чтобы изменить [[self]] внутри value type
    

---

### Пример 6. Enum с [[Result]]-like паттерном

```swift
enum NetworkResult {
    case success(data: String)
    case failure(error: Error)
}

func fetchData(completion: (NetworkResult) -> Void) {
    if Bool.random() {
        completion(.success(data: "Data loaded"))
    } else {
        completion(.failure(error: NSError(domain: "", code: -1)))
    }
}

fetchData { result in
    switch result {
    case .success(let data): print(data)
    case .failure(let error): print(error)
    }
}
```

- Очень часто используется для **обработки результатов и ошибок**
    

---

## 5. Особенности enum

1. **Value type** — всегда копируется при присваивании
    
2. **Raw value vs associated value** — raw value фиксировано, associated value хранит данные внутри case
    
3. **Switch statement** — идеально работает с enum для exhaustive проверки
    
4. **Методы и computed properties** — делают enum «умным» типом
    
5. **Mutating methods** — нужны для изменения self
    

---

## 6. Итог

- **Enum** = тип с фиксированными вариантами (cases)
    
- Может хранить **raw values, associated values, методы и свойства**
    
- Отлично подходит для **безопасного кода и обработки вариантов**
    
- Используется в networking, state management, UI, моделировании данных
    

---

## 7. Рекурсивные (indirect) enum и reference-подобное поведение

В Swift `enum` **никогда не становится reference type** (в отличие от `class`).  
Однако существует особый случай — **рекурсивные enum**, которые могут **косвенно содержать самих себя**.

Для этого используется ключевое слово `indirect`.

### Что такое indirect enum

`indirect` позволяет enum ссылаться на самого себя через ассоциированные значения.

```swift
enum List {
    case end
    indirect case node(Int, List)
}
```

Или так:

```swift
indirect enum List {
    case end
    case node(Int, List)
}
```

Такой enum используется для описания **деревьев, списков, графов, AST, состояний** и других рекурсивных структур.

---

### Почему кажется, что enum становится reference type

Под капотом Swift **хранит recursive case через ссылку**, иначе компилятор не смог бы определить размер типа.

Это даёт эффект, похожий на reference semantics:

```swift
var list1 = List.node(1, .node(2, .end))
var list2 = list1
```

Формально:

- `list1` и `list2` — **value type**
    
- но вложенные `indirect` части **могут шариться**
    

Swift применяет **Copy-on-Write**, и копирование происходит **только при мутации**.

---

### Важный момент (экзаменационная формулировка)

✅ Enum **не становится reference type**  
✅ Enum **остается value type**  
⚠️ Но `indirect enum` **использует ссылочное хранение внутренних данных**

Правильнее говорить:

> «Enum может иметь reference-подобное поведение при использовании `indirect`,  
> но семантически он остаётся value type».

---

### Пример: дерево выражений (AST)

```swift
indirect enum Expression {
    case number(Int)
    case add(Expression, Expression)
    case multiply(Expression, Expression)
}

let expr = Expression.add(
    .number(2),
    .multiply(.number(3), .number(4))
)
```

Такое **невозможно реализовать без indirect**, и именно здесь enum — идеальный инструмент.

---

### Когда использовать indirect enum

Используй `indirect enum`, если:

- есть **рекурсивная структура**
    
- нужен **строгий набор состояний**
    
- важна **exhaustive проверка switch**
    
- не хочется переходить на `class`
    

Типичные кейсы:

- парсеры
    
- AST
    
- state machine
    
- UI-навигация
    
- сложные бизнес-сценарии
    

---

### Итог по indirect enum

- `enum` **всегда value type**
    
- `indirect` разрешает рекурсию
    
- внутри используется **ссылочное хранение**
    
- поведение похоже на reference, но семантика остаётся value
    
- безопаснее и предсказуемее, чем `class`
    
