#swift #memory_management #copy_on_write #performance #value_type
# Copy-On-Shadow (COS) в Swift

**Copy-On-Shadow** — это техника **ленивой оптимизации памяти**, похожая на [[Copy-On-Write]] (COW), но с более мягким подходом к копированию.

### Главная идея COS

- При **присваивании** переменной создаётся **теневая ссылка** (shadow reference) на общий буфер — **без копирования**.
- Копия буфера создаётся **только при реальной записи** через эту теневую ссылку.
- **Нет** проверки `isKnownUniquelyReferenced` на каждом доступе → меньше overhead.

### Сравнение COW vs COS

| Характеристика           | Copy-On-Write (COW)                              | Copy-On-Shadow (COS)                                    |
| ------------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| Когда копируется         | При первом write, если есть другие ссылки        | Только при реальном изменении через shadow              |
| Проверка уникальности    | Да (`isKnownUniquelyReferenced`) на каждом write | Нет — копирование ленивое                               |
| Overhead на чтение       | Есть (проверка уникальности)                     | Минимальный                                             |
| Overhead на присваивание | Почти нулевой                                    | Почти нулевой                                           |
| Подходит для             | Стандартные коллекции [[Swift]]                  | Кастомные типы с частыми присваиваниями и редкими write |

### Пример реализации (упрощённая имитация COS)

```swift
final class ShadowStorage<T> {
    var buffer: [T]
    init(_ buffer: [T]) { self.buffer = buffer }
}

struct ShadowArray<T> {
    private var storage: ShadowStorage<T>
    
    init(_ elements: [T]) {
        storage = ShadowStorage(elements)
    }
    
    subscript(index: Int) -> T {
        get { storage.buffer[index] }
        set {
            // Копируем только при первой модификации
            if !isKnownUniquelyReferenced(&storage) {
                storage = ShadowStorage(storage.buffer)
            }
            storage.buffer[index] = newValue
        }
    }
    
    mutating func append(_ element: T) {
        if !isKnownUniquelyReferenced(&storage) {
            storage = ShadowStorage(storage.buffer)
        }
        storage.buffer.append(element)
    }
}

// Использование
var a = ShadowArray([1, 2, 3])
var b = a                // теневой reference — общий буфер

a[0] = 10                // копирование произошло только здесь
print(a[0])              // 10
print(b[0])              // 1 — b остался со старым буфером
```

### Реальный сценарий применения COS-подхода

```swift
// Большой массив конфигураций, который часто копируется, но редко меняется
struct ConfigSnapshot {
    private var storage: ShadowStorage<[String: Any]>
    
    init(_ dict: [String: Any]) {
        storage = ShadowStorage(dict)
    }
    
    subscript(key: String) -> Any? {
        get { storage[key] }
        set {
            if !isKnownUniquelyReferenced(&storage) {
                storage = ShadowStorage(storage.buffer)
            }
            storage[key] = newValue
        }
    }
}
```

### Когда COS может быть полезнее COW

- Очень **много присваиваний** (копирований) одной и той же коллекции
- **Чтение** происходит намного чаще, чем запись
- Коллекция **большая** (сотни тысяч — миллионы элементов)
- Хотим минимизировать проверки `isKnownUniquelyReferenced` на горячем пути

### Итог — коротко и честно (2026)

- **COW** — стандарт Swift для [[Array]], [[Dictionary]], [[Set]], [[String]]
- **COS** — более агрессивная ленивая оптимизация, которую можно реализовать вручную
- Встроенного COS в Swift **нет** — это кастомная техника
- Используй COS, когда профилирование показывает, что `isKnownUniquelyReferenced` съедает заметное время
- В 99% случаев **хватит стандартного COW** — он уже очень хорошо оптимизирован

**Короткое правило**:  
«COW — по умолчанию. COS — когда Instruments кричит, что COW слишком дорогой на чтение.»
