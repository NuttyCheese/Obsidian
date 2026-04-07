#swift #subscript #syntax #collection #custom-operator

---
### Определение
**Сабскрипты (subscripts)** — это сокращенный синтаксис для доступа к элементам коллекции, списка или последовательности через квадратные скобки `[]`. Они позволяют обращаться к экземпляру типа так, как будто это массив или словарь, предоставляя удобный интерфейс для получения и установки значений.

Сабскрипты не ограничиваются только коллекциями — вы можете реализовать их для любого типа, где имеет смысл доступ к значениям по индексу, ключу или другому параметру.

### Зачем это знать [[iOS]]-разработчику?
1.  **Удобство [[API]]:** Делает ваш тип интуитивно понятным (как [[Array]]`[index]` или [[Dictionary]]`[key]`).
2.  **Гибкость:** Может принимать любое количество параметров любого типа и возвращать любой тип.
3.  **Перегрузка:** Можно определить несколько сабскриптов для одного типа с разными сигнатурами.
4.  **Кастомные коллекции:** При создании своих коллекций или контейнеров сабскрипты — must-have.

---

### Базовый синтаксис

```swift
subscript(parameters) -> ReturnType {
    get {
        // Возвращаем значение
    }
    set(newValue) {
        // Устанавливаем значение (параметр newValue опционален)
    }
}
```

- **[[getter|get]]** — обязателен, если сабскрипт только для чтения.
- **[[setter|set]]** — опционален, делает сабскрипт доступным для записи.

---

### Примеры использования

#### 1. **Сабскрипт только для чтения**

```swift
struct TimesTable {
    let multiplier: Int
    
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}

let threeTimesTable = TimesTable(multiplier: 3)
print(threeTimesTable[6])  // 18
```

#### 2. **Сабскрипт с get и set**

```swift
class ShoppingList {
    private var items: [String] = []
    
    subscript(index: Int) -> String {
        get {
            return items[index]
        }
        set {
            items[index] = newValue
        }
    }
    
    func add(_ item: String) {
        items.append(item)
    }
}

var list = ShoppingList()
list.add("Milk")
list.add("Bread")
print(list[0])   // Milk
list[1] = "Butter"
print(list[1])   // Butter
```

#### 3. **Сабскрипт с несколькими параметрами**

```swift
struct Matrix {
    let rows: Int
    let columns: Int
    private var grid: [Double]
    
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        self.grid = Array(repeating: 0.0, count: rows * columns)
    }
    
    subscript(row: Int, column: Int) -> Double {
        get {
            return grid[(row * columns) + column]
        }
        set {
            grid[(row * columns) + column] = newValue
        }
    }
}

var matrix = Matrix(rows: 2, columns: 2)
matrix[0, 1] = 5.5
print(matrix[0, 1])  // 5.5
```

#### 4. **Сабскрипт с разными типами параметров**

```swift
struct DataStore {
    private var intStore: [Int: String] = [:]
    private var stringStore: [String: String] = [:]
    
    subscript(key: Int) -> String? {
        get { intStore[key] }
        set { intStore[key] = newValue }
    }
    
    subscript(key: String) -> String? {
        get { stringStore[key] }
        set { stringStore[key] = newValue }
    }
}

var store = DataStore()
store[42] = "Answer"
store["greeting"] = "Hello"

print(store[42] ?? "")        // Answer
print(store["greeting"] ?? "") // Hello
```

#### 5. **Сабскрипт с вариативными параметрами**

```swift
struct Path {
    subscript(pathComponents: String...) -> String {
        return "/" + pathComponents.joined(separator: "/")
    }
}

let path = Path()
print(path["users", "john", "profile"])  // /users/john/profile
```

---

### Сабскрипты в типах

#### 1. **В классах**

```swift
class DataCache {
    private var cache: [String: Data] = [:]
    
    subscript(key: String) -> Data? {
        get { cache[key] }
        set { cache[key] = newValue }
    }
}
```

#### 2. **В структурах (мутация)**

```swift
struct ColorPalette {
    private var colors: [String] = ["Red", "Green", "Blue"]
    
    subscript(index: Int) -> String {
        get { colors[index] }
        set(newColor) {
            colors[index] = newColor
        }
    }
}

var palette = ColorPalette()
palette[0] = "Crimson"
print(palette[0])  // Crimson
```

#### 3. **В перечислениях**

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars
    
    subscript(orbit: Int) -> String {
        switch orbit {
        case 1: return "Mercury"
        case 2: return "Venus"
        case 3: return "Earth"
        case 4: return "Mars"
        default: return "Unknown"
        }
    }
}

print(Planet.earth[3])  // Earth
```

---

### Статические сабскрипты

Можно определить сабскрипт на уровне типа (с модификатором [[static]]).

```swift
enum AppConfig {
    static subscript(key: String) -> String? {
        return Bundle.main.object(forInfoDictionaryKey: key) as? String
    }
}

let version = AppConfig["CFBundleShortVersionString"]
print(version ?? "")  // 1.0
```

---

### Сабскрипты и опциональность

Сабскрипты могут возвращать опциональные значения для безопасного доступа.

```swift
struct SafeArray<T> {
    private var elements: [T] = []
    
    subscript(index: Int) -> T? {
        guard index >= 0 && index < elements.count else { return nil }
        return elements[index]
    }
    
    mutating func append(_ element: T) {
        elements.append(element)
    }
}

var safe = SafeArray<String>()
safe.append("A")
safe.append("B")

print(safe[0] ?? "none")  // A
print(safe[5] ?? "none")  // none
```

---

### Сабскрипты в расширениях

Можно добавлять сабскрипты в существующие типы через расширения.

```swift
extension String {
    subscript(safe index: Int) -> Character? {
        guard index >= 0 && index < count else { return nil }
        return self[self.index(startIndex, offsetBy: index)]
    }
}

let text = "Hello"
print(text[safe: 1] ?? "")  // e
print(text[safe: 10] ?? "") // (пусто)
```

---

### Сабскрипт с ключевым словом `dynamicMemberLookup`

Сабскрипты также используются в `@dynamicMemberLookup` для динамического доступа к свойствам.

```swift
@dynamicMemberLookup
struct JSON {
    private var value: Any
    
    init(_ value: Any) {
        self.value = value
    }
    
    subscript(dynamicMember member: String) -> JSON {
        let dict = value as? [String: Any] ?? [:]
        return JSON(dict[member] as Any)
    }
    
    subscript(index: Int) -> JSON {
        let array = value as? [Any] ?? []
        return JSON(array.indices.contains(index) ? array[index] : NSNull())
    }
}

let json = JSON(["user": ["name": "Alice", "age": 30]])
print(json.user.name.value)  // Alice
```

---

### Лучшие практики

#### 1. **Документируйте параметры и возвращаемое значение**

```swift
/// Доступ к элементу по индексу.
/// - Parameter index: Позиция элемента.
/// - Returns: Элемент или nil, если индекс вне диапазона.
subscript(index: Int) -> Element? { ... }
```

#### 2. **Используйте осмысленные имена параметров**

```swift
// ✅ Хорошо
subscript(row: Int, column: Int) -> Double

// ❌ Плохо
subscript(a: Int, b: Int) -> Double
```

#### 3. **Не делайте сабскрипты слишком сложными**

Сабскрипты должны быть интуитивно понятными. Если логика сложная, лучше использовать именованные методы.

```swift
// ❌ Плохо — непонятно
subscript(a: String, b: Int, c: Bool) -> Result

// ✅ Хорошо — ясное имя
func fetchData(for key: String, page: Int, forceReload: Bool) -> Result
```

#### 4. **Поддерживайте безопасность через опциональные возвраты**

```swift
// ✅ Безопасно
subscript(index: Int) -> Element? { ... }

// ❌ Опасно (может упасть)
subscript(index: Int) -> Element { ... }
```

---

### Ограничения

1.  **Нельзя использовать [[inout]] параметры**.
2.  **Нельзя иметь несколько вариативных параметров**.
3.  **Сабскрипты не могут быть `mutating` для классов (только для структур и перечислений)**.

---

### Короткое правило

> **Сабскрипты** позволяют обращаться к экземпляру через квадратные скобки.  
> Используйте их для коллекций, матриц, кэшей и любых типов, где имеет смысл доступ по индексу или ключу.

### Итог

**Сабскрипты** в Swift:

1.  **Обеспечивают удобный синтаксис** доступа к элементам.
2.  **Могут быть только для чтения** (только `get`) или для чтения и записи (`get` + `set`).
3.  **Поддерживают перегрузку** (разные типы параметров).
4.  **Могут быть статическими** (`static subscript`).
5.  **Используются в `@dynamicMemberLookup`** для динамических свойств.

Понимание сабскриптов помогает создавать чистые и интуитивно понятные API, особенно при работе с коллекциями и кастомными контейнерами.