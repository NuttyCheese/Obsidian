**Numeric** в [[Swift]] — это протокол, введённый в Swift 5.7 (2022 год) и активно развиваемый в Swift 6 (2024–2026). Он объединяет все встроенные числовые типы, которые поддерживают базовые арифметические операции, сравнение и другие числовые возможности.

### Основные типы, conforming to Numeric (2026 актуальные)

| Тип                            | Signed? | Integer? | Floating-point?  | Битность (64-битная система) | Самый частый сценарий          |
| ------------------------------ | ------- | -------- | ---------------- | ---------------------------- | ------------------------------ |
| [[Int]] / `UInt`               | ±       | Да       | Нет              | 64 бита                      | Счётчики, индексы              |
| `Int8` / `UInt8`               | ±       | Да       | Нет              | 8 бит                        | Байты, цвета, флаги            |
| `Int16` / `UInt16`             | ±       | Да       | Нет              | 16 бит                       | Аудио/видео сэмплы             |
| `Int32` / `UInt32`             | ±       | Да       | Нет              | 32 бита                      | OpenGL, Core Graphics          |
| `Int64` / `UInt64`             | ±       | Да       | Нет              | 64 бита                      | ID, timestamps                 |
| [[Float]]                      | ±       | Нет      | Да               | 32 бита                      | Графика, UI ([[CGFloat]])      |
| [[Double]]                     | ±       | Нет      | Да               | 64 бита                      | Научные вычисления, координаты |
| `Float16` (iOS 16+, macOS 13+) | ±       | Нет      | Да               | 16 бит                       | ML, графика (экономия памяти)  |
| `Decimal`                      | ±       | Нет      | Да ([[Decimal]]) | 128 бит                      | Финансы, точные расчёты        |

### Ключевые протоколы, связанные с Numeric (иерархия 2026)

```swift
Numeric  ← SignedNumeric  ← SignedInteger / FloatingPoint / Decimal
         ← AdditiveArithmetic
         ← ExpressibleByIntegerLiteral / ExpressibleByFloatLiteral
         ← Comparable (не всегда)
```

### Самые полезные возможности Numeric в 2026 году

1. **Обобщённые функции для любых чисел**

```swift
func sum<T: Numeric>(_ numbers: [T]) -> T {
    numbers.reduce(0, +)
}

let ints = [1, 2, 3]
print(sum(ints))           // 6 (Int)

let doubles = [1.5, 2.5, 3.5]
print(sum(doubles))        // 7.5 (Double)

let floats: [Float] = [1.1, 2.2]
print(sum(floats))         // 3.3 (Float)
```

2. **Обобщённая математика**

```swift
func square<T: Numeric>(_ x: T) -> T {
    x * x
}

print(square(5))           // 25 (Int)
print(square(3.14))        // ~9.8596 (Double)
```

3. **Numeric в протоколах и generics**

```swift
protocol Scalable {
    associatedtype Magnitude: Numeric
    var magnitude: Magnitude { get }
}

struct Vector2D<T: Numeric> {
    let x: T
    let y: T
    
    var magnitude: T {
        (x * x + y * y).squareRoot() as? T ?? 0  // требует FloatingPoint
    }
}
```

### Лучшие практики работы с Numeric в Swift 2026

- **Используй Numeric**, когда нужна **обобщённая** работа с числами (сумма, умножение, среднее и т.д.)  
- **SignedNumeric** — если нужен знак (+/-)  
- **FloatingPoint** — если нужны дробные числа и sqrt/pow/trigonometry  
- **BinaryInteger** — если работаешь только с целыми числами  
- **Decimal** — **единственный** выбор для финансовых расчётов (никогда не используй [[Double]]/[[Float]] для денег)  
- **Float16** — для ML и графики (экономия памяти на A17 Pro/M3+)  
- **Swift 6 strict concurrency** — Numeric-типы **Sendable** — безопасны для передачи между задачами  
- **Избегай NSNumber** — приводи к Swift-примитивам сразу  
- **Документируйте** — пиши комментарий «Numeric — обобщённый числовой тип»

**Короткий девиз 2026**:
> «Numeric — это когда ты хочешь написать одну функцию, которая работает с [[Int]], Double, Float и [[UInt]] одновременно.  
> В 2026 году это **основной** способ писать обобщённую математику в Swift.  
> Для финансов — всегда Decimal, для графики — CGFloat/Double, для индексов — Int.»
