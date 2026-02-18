**MemoryLayout** в [[Swift]] — это встроенный тип ([[generic]] [[struct]]), который позволяет получать информацию о **размере**, **выравнивании** и **смещении** (stride, alignment, offset) любого типа в памяти.

Это один из самых полезных инструментов низкоуровневого программирования в Swift, особенно когда нужно:

- работать с сырыми буферами (UnsafeRawBufferPointer, [[Data]], C-структурами)  
- оптимизировать память (padding, alignment)  
- взаимодействовать с C/C++/[[Objective-C]] [[API]]  
- писать высокопроизводительный код (графика, аудио, нейросети, парсинг бинарных форматов)

### Основные статические свойства MemoryLayout (2026 актуальные)

```swift
MemoryLayout<T>.size      // Размер типа в байтах (без padding'а в массивах/структурах)
MemoryLayout<T>.stride    // Размер одного элемента с учётом выравнивания (чаще всего используется)
MemoryLayout<T>.alignment // Требуемое выравнивание типа в памяти (обычно степень двойки: 1, 2, 4, 8, 16…)
```

### Примеры использования MemoryLayout (самые частые сценарии 2026)

#### 1. Размер и stride встроенных типов

```swift
print(MemoryLayout<Int>.size)       // 8 байт на 64-битных системах
print(MemoryLayout<Int>.stride)     // 8 байт (то же)
print(MemoryLayout<Int>.alignment)  // 8 байт

print(MemoryLayout<Bool>.size)      // 1 байт
print(MemoryLayout<Bool>.stride)    // 1 байт
print(MemoryLayout<Bool>.alignment) // 1 байт

print(MemoryLayout<CGPoint>.size)   // 16 байт (два CGFloat = 8 + 8)
print(MemoryLayout<CGPoint>.stride) // 16 байт
print(MemoryLayout<CGPoint>.alignment) // 8 байт
```

#### 2. Размер и stride в структурах (padding и alignment — самое важное!)

```swift
struct Point3D {
    let x: Float    // 4 байта
    let y: Float    // 4 байта
    let z: Float    // 4 байта
    // padding 4 байта → выравнивание под 8 байт (CGFloat/Float64)
}

print(MemoryLayout<Point3D>.size)     // 12 байт (без padding)
print(MemoryLayout<Point3D>.stride)   // 16 байт (с padding'ом)
print(MemoryLayout<Point3D>.alignment) // 4 байта

struct OptimizedPoint3D {
    let x: Float
    let y: Float
    let z: Float
    let padding: UInt32 = 0  // явно добавляем padding
}

print(MemoryLayout<OptimizedPoint3D>.stride) // 16 байт (но без скрытого padding)
```

#### 3. Работа с UnsafeRawBufferPointer / Data (очень частый сценарий)

```swift
let data = Data([0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08])

data.withUnsafeBytes { buffer in
    let intValue = buffer.load(as: UInt64.self)
    print("Первые 8 байт как UInt64:", intValue)
    
    // Смещение на 4 байта
    let offset = MemoryLayout<UInt32>.stride
    let uint32Value = buffer.load(fromByteOffset: offset, as: UInt32.self)
    print("Со смещением \(offset) байт как UInt32:", uint32Value)
}
```

#### 4. Проверка выравнивания перед load(fromByteOffset:as:)

```swift
func safelyLoad<T>(from buffer: UnsafeRawBufferPointer, offset: Int) -> T? {
    guard offset + MemoryLayout<T>.size <= buffer.count,
          offset % MemoryLayout<T>.alignment == 0 else {
        return nil
    }
    return buffer.load(fromByteOffset: offset, as: T.self)
}
```

### Лучшие практики MemoryLayout в Swift 2026

- **Всегда используй stride, а не size** — когда работаешь с массивами/буферами  
  → `stride` учитывает padding между элементами  
  → `size` — только чистый размер типа

- **Проверяй alignment перед load/store** — иначе undefined behavior (краш или мусор)  
- **UnsafeRawBufferPointer / UnsafeMutableRawBufferPointer** — чаще всего используется с MemoryLayout  
- **Swift 6 strict concurrency** — `MemoryLayout` сам по себе thread-safe, но указатели (Unsafe*) требуют осторожности  
- **Не полагайся на размер типов** в разных архитектурах  
  → `Int` = 8 байт на 64-бит, но `CGFloat` = 4 байта на 32-бит (редко, но важно)  
- **Документируйте** — пиши комментарий «MemoryLayout<UInt64>.stride — размер с выравниванием»

**Короткий девиз 2026**:
> «MemoryLayout — это когда тебе нужно точно знать, сколько байт занимает тип в памяти и как он выравнивается.  
> В 2026 году это must-have для работы с Unsafe-указателями, бинарными форматами, C-API и высокопроизводительным кодом.  
> Главное правило: используй .stride, а не .size, когда работаешь с массивами и буферами.»

Удачи с низкоуровневой, но безопасной работой с памятью в Swift! 🧠