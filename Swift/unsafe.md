**`unsafe`** — это ключевое слово и концепция, которая обозначает **операции, которые могут нарушить безопасность памяти**.

- Используется для **низкоуровневого доступа к памяти**, указателям, C [[API]] и оптимизации
    
- `unsafe` **не проверяет границы и корректность** — ответственность за правильность лежит на разработчике
    
- В [[Swift]] есть несколько `unsafe` типов и функций:
    
    - `UnsafePointer` / `UnsafeMutablePointer`
        
    - `UnsafeBufferPointer`
        
    - `UnsafeRawPointer` / `UnsafeMutableRawPointer`
        
    - `withUnsafePointer(to:)` / `withUnsafeMutablePointer(to:)`
        

> Проще говоря: `unsafe` = «работа напрямую с памятью, без гарантий безопасности».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UnsafePointer**|Неизменяемый указатель на память типа T|
|**UnsafeMutablePointer**|Изменяемый указатель на память типа T|
|**UnsafeRawPointer**|Сырой указатель на память, без типа|
|**UnsafeMutableRawPointer**|Сырой изменяемый указатель|
|**withUnsafePointer(to:)**|Передаёт безопасно указатель на значение|
|**withUnsafeBytes**|Позволяет работать с байтами памяти объекта|
|**Memory layout**|Структура данных в памяти (`MemoryLayout<T>`)|

---

## 3. Основной синтаксис

```swift
var number: Int = 42
withUnsafePointer(to: &number) { pointer in
    print(pointer.pointee) // 42
}
```

- `&number` передаёт адрес переменной
    
- `pointee` — доступ к значению через указатель
    

---

## 4. Примеры от простого к сложному

### Пример 1. UnsafePointer

```swift
var value = 10
let pointer: UnsafePointer<Int> = withUnsafePointer(to: &value) { $0 }
print(pointer.pointee) // 10
```

- Ссылается на память переменной без возможности её менять
    

---

### Пример 2. UnsafeMutablePointer

```swift
var value = 5
withUnsafeMutablePointer(to: &value) { pointer in
    pointer.pointee += 10
}
print(value) // 15
```

- Позволяет **изменять значение через указатель**
    

---

### Пример 3. UnsafeRawPointer и UnsafeMutableRawPointer

```swift
var x: UInt32 = 123
withUnsafeBytes(of: &x) { rawPointer in
    let byteArray = rawPointer.map { $0 }
    print(byteArray) // [123, 0, 0, 0] (little-endian)
}
```

- Позволяет работать с **сырыми байтами памяти**
    

---

### Пример 4. UnsafeBufferPointer

```swift
var array = [1, 2, 3, 4]
array.withUnsafeBufferPointer { buffer in
    for i in 0..<buffer.count {
        print(buffer[i])
    }
}
```

- Предоставляет безопасный способ **итерации по памяти массива**
    

---

### Пример 5. Использование с C API

```swift
import Foundation

var cString = "Hello".utf8CString
cString.withUnsafeMutableBufferPointer { buffer in
    let ptr = buffer.baseAddress!
    print(String(cString: ptr)) // Hello
}
```

- Для взаимодействия с **C функциями** нужен прямой доступ к памяти
    

---

## 5. Особенности unsafe

1. **Не проверяет границы** — ответственность за корректность лежит на разработчике
    
2. Может **повлиять на производительность**, но небезопасно
    
3. Используется для **низкоуровневой работы с памятью, C API, оптимизаций**
    
4. Сохраняет возможность Swift **структур безопасности**, если использовать с `withUnsafe...` блоками
    

---

## 6. Итог

- **unsafe** = низкоуровневый доступ к памяти
    
- Используется с **указателями и сырыми байтами**
    
- Требует внимательности — ошибки могут привести к **crash, data corruption**
    
- Используется для **высокой производительности и взаимодействия с C API**
    

---
