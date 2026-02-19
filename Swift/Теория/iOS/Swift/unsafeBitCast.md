**`unsafeBitCast`** — это низкоуровневая функция в стандартной библиотеке [[Swift]], которая выполняет **битовое переинтерпретацию** значения одного типа как значение другого типа **без проверки совместимости размеров и выравнивания**.

```swift
func unsafeBitCast<T, U>(_ x: T, to: U.Type) -> U
```

### Главные характеристики unsafeBitCast (2026 актуально)

| Характеристика                 | Описание                                                              | Последствия при неправильном использовании    |
| ------------------------------ | --------------------------------------------------------------------- | --------------------------------------------- |
| **Не проверяет размер типов**  | `MemoryLayout<T>.size == MemoryLayout<U>.size` не гарантируется       | Чтение мусора, переполнение, UB               |
| **Не проверяет выравнивание**  | Может нарушить требования alignment                                   | Краш на некоторых архитектурах (особенно ARM) |
| **Не проверяет совместимость** | Можно привести [[Double]] к [[Int]], `UnsafePointer` к `Int64` и т.д. | Непредсказуемое поведение                     |
| **Не сохраняет семантику**     | Битовая копия, а не значение                                          | [[Float]] → `Int` не равно округлению         |
| **Полностью unsafe**           | Отключает почти все гарантии безопасности Swift                       | Ответственность 100% на разработчике          |

### Когда unsafeBitCast реально нужен (редкие, но оправданные случаи)

| Сценарий                                                 | Почему именно unsafeBitCast                         | Альтернатива (когда можно обойтись без него) |
| -------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------- |
| Преобразование указателя в целое число (pointer tagging) | Хранение метаданных в младших битах указателя       | `Int(bitPattern:)` / `UInt(bitPattern:)`     |
| Работа с union-подобными структурами в C-[[API]]         | C-структуры с union                                 | `UnsafeMutableRawPointer` + `load(as:)`      |
| Быстрое чтение битового представления числа              | Доступ к знаковому биту `Float` / `Double`          | `Float.bitPattern` / `Double.bitPattern`     |
| Низкоуровневая оптимизация (SIMD, [[Metal]], Accelerate) | Переинтерпретация векторов/буферов                  | `UnsafeRawBufferPointer.load(as:)`           |
| Обход ABI-ограничений в старом [[Objective-C]] коде      | Редкие случаи совместимости со старыми фреймворками | —                                            |

### Самые безопасные и рекомендуемые альтернативы unsafeBitCast

| Задача                                              | Рекомендуемый современный способ (2026)                     | Почему лучше unsafeBitCast |
|-----------------------------------------------------|-------------------------------------------------------------|-----------------------------|
| Указатель → целое число                             | `Int(bitPattern: pointer)` / `UInt(bitPattern: pointer)`    | Типобезопасно, читаемо      |
| Число → битовая маска                               | `Float.bitPattern` / `Double.bitPattern`                    | Специально предназначено    |
| Чтение произвольных данных из буфера                | `buffer.load(as: T.self)` / `load(fromByteOffset:as:)`      | Проверяет границы           |
| Преобразование между совместимыми типами            | `withMemoryRebound` или `assumingMemoryBound(to:)`          | Сохраняет проверку типа     |
| Работа с union в C-структурах                       | `UnsafeMutableRawPointer.bindMemory(to:)` + `load(as:)`     | Более контролируемо         |

### Примеры реального (и безопасного) использования unsafeBitCast (очень редкие случаи)

#### 1. Классический pointer tagging (хранение метаданных в указателе)

```swift
func tagPointer<T>(_ ptr: UnsafeMutableRawPointer, tag: Int) -> UnsafeMutableRawPointer {
    let tagged = Int(bitPattern: ptr) | tag
    return UnsafeMutableRawPointer(bitPattern: tagged)!
}

// Обратное извлечение
func getTag(from ptr: UnsafeRawPointer) -> Int {
    return Int(bitPattern: ptr) & 0xF // последние 4 бита как тег
}
```

Здесь `unsafeBitCast` не нужен — лучше `Int(bitPattern:)` и `UnsafeRawPointer(bitPattern:)`.

#### 2. Чтение битового представления (почти никогда не нужно unsafeBitCast)

```swift
let f: Float = 3.14
let bits = f.bitPattern          // UInt32 — правильно
let wrong = unsafeBitCast(f, to: UInt32.self)  // тоже работает, но хуже
```

### 6. Почему unsafeBitCast почти исчез в современном коде

- Появились **типобезопасные альтернативы**:
  - `Int(bitPattern:)`, `UInt(bitPattern:)`
  - `Float/Double.bitPattern`
  - `UnsafeRawBufferPointer.load(as:)`
  - `withMemoryRebound`
- Компилятор стал строже (Swift 5.7+ strict concurrency, Swift 6)
- Большинство низкоуровневых задач теперь решаются через `UnsafeBufferPointer` / `UnsafeRawBufferPointer`
- Apple активно рекомендует избегать `unsafeBitCast` в документации и WWDC

### 7. Рекомендации 2026 года

- **Никогда** не используйте `unsafeBitCast` в новом коде, если есть альтернатива  
- **Предпочитайте**:
  - `Int(bitPattern:)`, `UInt(bitPattern:)`
  - `Float/Double.bitPattern`
  - `UnsafeRawBufferPointer.load(as:)`
  - `withMemoryRebound(to:capacity:)`  
- **Если всё-таки нужен unsafeBitCast** — обязательно документируйте:
  ```swift
  // unsafeBitCast: битовая переинтерпретация Double → UInt64 для доступа к знаковому биту
  let bits = unsafeBitCast(value, to: UInt64.self)
  ```
- **Swift 6 strict concurrency** — `unsafeBitCast` безопасен, но результат должен быть `Sendable` при передаче между акторами  
- **Тестируйте** на разных архитектурах (x86_64, arm64, arm64e) — поведение может отличаться при нарушении alignment

**Короткий девиз 2026**:
> `unsafeBitCast` — это **последний** инструмент в ящике, когда все безопасные способы уже испробованы и не подошли.  
> В 2026 году почти всегда есть лучший вариант:  
> - `Int/UInt(bitPattern:)`  
> - `Float/Double.bitPattern`  
> - `UnsafeRawBufferPointer.load(as:)`  
> - `withMemoryRebound`  
> Используйте `unsafeBitCast` только если **точно понимаете**, что делаете, и **документируйте** это.
