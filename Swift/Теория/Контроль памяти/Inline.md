**Inline** (встроенное хранение) — когда данные хранятся **не по указателю** в отдельном объекте на куче, а **непосредственно внутри** другой структуры, объекта или контейнера.

### Основные случаи inline

| Случай                                   | Где хранится                          | Пример кода                                    | Комментарий                        |
| ---------------------------------------- | ------------------------------------- | ---------------------------------------------- | ---------------------------------- |
| [[struct]] внутри struct                 | Внутри памяти родительской структуры  | `struct Rect { var p1: Point; var p2: Point }` | Полностью inline                   |
| struct внутри [[class]]                  | Внутри памяти экземпляра класса       | `class Box { var point = Point(x:0,y:0) }`     | Inline внутри heap-объекта         |
| Маленький struct в `any` [[Protocol]]    | В value buffer existential-контейнера | `let p: any P = SmallStruct()`                 | ≤ 3 слова — inline, иначе [[Heap]] |
| Маленькие элементы в [[Array]]           | В contiguous буфере массива           | `let arr = [Point](repeating: …, count: 100)`  | Inline в одном буфере              |
| Associated values в [[enum]] (маленькие) | Внутри памяти enum                    | `enum Result { case success(Point) }`          | Inline, если помещается            |

### Inline vs Heap vs Stack (коротко)

| Тип хранения  | Где физически                | Пример                             | Скорость доступа | Управление памятью |
| ------------- | ---------------------------- | ---------------------------------- | ---------------- | ------------------ |
| **Inline**    | Внутри родителя              | `struct` внутри `class` / `Array`  | Максимальная     | Автоматическое     |
| **[[Heap]]**  | Отдельный объект (ARC)       | `class`, замыкания, большие struct | Медленнее        | [[ARC]]            |
| **[[Stack]]** | Локальная переменная функции | `let x = 42`, маленькие struct     | Самая быстрая    | Автоматическое     |

### Почему inline важен

- **Меньше аллокаций** на куче
- **Лучшая локальность данных** → CPU cache friendly
- **Меньше косвенных обращений** (pointer chasing)
- **Экономия** при работе с коллекциями и протоколами (`any`)

### Примеры на практике

```swift
// 1. Полностью inline
struct Point { var x, y: Int }
struct Rect  { var topLeft: Point; var bottomRight: Point }

var r = Rect(topLeft: Point(x:0,y:0), bottomRight: Point(x:10,y:10))
// всё лежит в одном contiguous блоке памяти
```

```swift
// 2. Inline внутри класса
class Container {
    var point = Point(x: 5, y: 7)     // inline внутри объекта Container
}
```

```swift
// 3. Inline в existential (any Protocol)
protocol Drawable { func draw() }
struct Circle: Drawable { let radius: Int; func draw() { … } }

let shape: any Drawable = Circle(radius: 10)
// Circle (маленький) → inline в value buffer
// Большой struct → вынесен на heap
```

### Итог — коротко

- **Inline** = данные лежат **внутри** родительской структуры/объекта
- **Преимущества**: быстрее доступ, меньше аллокаций, лучше кэш
- **Типичные места**:
  - struct внутри [[struct]]/[[class]]
  - Маленькие типы в `any Protocol`
  - Элементы в [[Array]] / `ContiguousArray`
- **Когда не inline**: большие struct в [[any]], объекты `class`, замыкания

**Главное правило**:
> Если тип маленький и часто используется внутри других типов — компилятор старается держать его **inline**.  
> Это одна из причин, почему Swift такой быстрый.
