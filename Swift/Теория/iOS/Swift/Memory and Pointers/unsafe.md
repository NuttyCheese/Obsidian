**`unsafe`** в [[Swift]] — это **не ключевое слово**, а **концепция и префикс** в названиях типов и методов, которые позволяют **обходить гарантии безопасности памяти** языка.

Это **низкоуровневый доступ** к памяти, указателям и сырым данным, где компилятор **не проверяет** границы массивов, валидность указателей, переполнения и другие ошибки.  
Вся ответственность за корректность ложится **на разработчика**.

### Когда unsafe необходим (реальные случаи 2025–2026)

| Сценарий                                                                            | Почему нужен unsafe                                              | Альтернатива (когда можно обойтись без unsafe) |
| ----------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------- |
| Вызов C / C++ / [[Objective-C]] [[API]]                                             | Передача указателей (`UnsafeMutablePointer`, `UnsafeRawPointer`) | Swift-обёртки (если есть)                      |
| Парсинг бинарных форматов (аудио, видео, изображения, сетевые пакеты)               | Прямой доступ к байтам через `withUnsafeBytes`                   | `Data` + высокоуровневые методы                |
| Высокопроизводительная обработка массивов (Metal, [[Accelerate]], SIMD, Core Image) | Доступ к памяти без копирования                                  | `Array` / `ContiguousArray`                    |
| Копирование больших блоков памяти (`memcpy`)                                        | Максимальная скорость                                            | `Array` + `append(contentsOf:)`                |
| Работа с сырыми буферами из `Data`, `UnsafeBufferPointer`                           | Чтение/запись байт без промежуточных копий                       | —                                              |
| Интероперабельность с низкоуровневыми API (AVFoundation, Core Graphics, Metal)      | Требуются указатели                                              | Swift-обёртки (если есть)                      |

### Основные unsafe-типы и методы (актуальные в 2026)

| Тип / Метод                                      | Что даёт                                                 | Самый безопасный способ использования          | Примечание |
|--------------------------------------------------|----------------------------------------------------------|------------------------------------------------|------------|
| `UnsafePointer<T>`                               | Неизменяемый указатель на T                              | `withUnsafePointer(to: &var) { ptr in … }`     | Только чтение |
| `UnsafeMutablePointer<T>`                        | Изменяемый указатель на T                                | `withUnsafeMutablePointer(to: &var) { … }`     | Запись |
| `UnsafeRawPointer`                               | Сырой неизменяемый указатель на байты                    | `withUnsafeBytes(of: &var) { … }`              | Чтение байт |
| `UnsafeMutableRawPointer`                        | Сырой изменяемый указатель на байты                      | `withUnsafeMutableBytes(of: &var) { … }`       | Запись байт |
| `UnsafeBufferPointer<T>`                         | Неизменяемый срез массива как указатель                  | `array.withUnsafeBufferPointer { … }`          | Самый популярный |
| `UnsafeMutableBufferPointer<T>`                  | Изменяемый срез массива                                  | `array.withUnsafeMutableBufferPointer { … }`   | Для модификации |
| `UnsafeMutableRawBufferPointer`                  | Изменяемый буфер байт (часто из `Data`)                  | `data.withUnsafeMutableBytes { … }`            | Для C-API |

### Самые популярные и безопасные паттерны 2026

#### 1. Самый частый и рекомендуемый способ — `withUnsafeBufferPointer`

```swift
let numbers = [1, 2, 3, 4, 5]

numbers.withUnsafeBufferPointer { buffer in
    // buffer: UnsafeBufferPointer<Int>
    print(buffer.count)          // 5
    print(buffer[0])             // 1
    let sum = buffer.reduce(0, +)
    print(sum)                   // 15
}
```

#### 2. Изменение массива через `withUnsafeMutableBufferPointer`

```swift
var bytes: [UInt8] = [0x01, 0x02, 0x03, 0x04]

bytes.withUnsafeMutableBufferPointer { buffer in
    buffer[0] = 0xFF
    buffer.swapAt(1, 2)          // меняет 0x02 и 0x03 местами
}

print(bytes) // [255, 3, 2, 4]
```

#### 3. Парсинг бинарных данных (самый частый unsafe-кейс)

```swift
let data = Data([0x78, 0x56, 0x34, 0x12]) // Little-endian UInt32 = 0x12345678

data.withUnsafeBytes { buffer in
    guard buffer.count >= MemoryLayout<UInt32>.size else { return }
    
    let value = buffer.load(as: UInt32.self)
    print(String(format: "0x%08X", value)) // 0x12345678
}
```

#### 4. Передача в C-API (очень часто)

```swift
var status: OSStatus = 0

AudioObjectGetPropertyDataSize(
    AudioObjectID(kAudioObjectSystemObject),
    &address,                       // UnsafeMutablePointer<AudioObjectPropertyAddress>
    0,
    nil,
    &size
)
```

### 5. Правила безопасности при работе с unsafe (критически важно)

- **Никогда** не сохраняйте `UnsafePointer` / `UnsafeRawPointer` за пределами `withUnsafe…` блока  
  → после выхода из замыкания память может быть переиспользована → UB (undefined behavior)

- **Всегда** проверяйте границы перед `load(fromByteOffset:)`  
  ```swift
  guard offset + MemoryLayout<T>.size <= buffer.count else {
      fatalError("Out of bounds")
  }
  ```

- **Используйте** `MemoryLayout<T>.stride` при работе с массивами (учитывает padding)  
- **Не храните** unsafe-указатели в свойствах классов — это почти всегда ошибка  
- **Swift 6 strict concurrency** — все unsafe-типы **не [[Sendable]]** → работайте с ними в одной задаче или используйте `nonisolated(unsafe)`  
- **Документируйте** — пишите комментарий «withUnsafeMutableBufferPointer — низкоуровневая модификация байтов для C-API»

**Короткий девиз 2026**:
> `unsafe` — это когда Swift снимает с себя ответственность за безопасность памяти и говорит: «теперь ты сам за всё отвечаешь».  
> В 2026 году:  
> - основной инструмент — `withUnsafeBufferPointer` / `withUnsafeMutableBufferPointer`  
> - `withUnsafeBytes` / `withUnsafeMutableBytes` — для сырых байтов  
> - указатели держите **только внутри** withUnsafe-блока  
> - используйте `unsafe` только когда без него **невозможно** (C-API, Metal, бинарные форматы)  
