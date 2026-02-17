**Mirror** — это встроенный в Swift механизм **рефлексии** (reflection), который позволяет во время выполнения программы ([[Runtime]]) получать информацию о:

- типе объекта  
- его свойствах (имя + значение)  
- суперклассах  
- количестве и типах детей (children)  
- структуре (enum case, tuple labels и т.д.)  
- отображении объекта в виде коллекции пар (label, value)

Основные сценарии использования Mirror в 2026 году:

- Отладка и логирование сложных объектов  
- Автоматическая сериализация / десериализация (когда [[Codable]] не подходит)  
- Генерация UI-форм / инспекторов свойств (например, в дебаггере)  
- Тестирование и мокинг  
- Инструменты анализа (например, diff объектов)  
- Динамическая работа с данными (например, в [[NoSQL]]-подобных хранилищах)

**Важно**: Mirror — это **не полноценная рефлексия** как в Java/Python/Ruby.  
[[Swift]] сознательно ограничивает возможности Mirror ради безопасности, производительности и защиты от злоупотреблений.

### 2. Основные свойства и методы Mirror

| Свойство / Метод   | Тип                    | Что возвращает / делает                                  | Пример использования                    |
| ------------------ | ---------------------- | -------------------------------------------------------- | --------------------------------------- |
| `subjectType`      | `Any.Type`             | Тип отражаемого объекта                                  | `mirror.subjectType`                    |
| `children`         | `Children` (коллекция) | Коллекция пар `(label: String?, value: Any)`             | `for (label, value) in mirror.children` |
| `displayStyle`     | `DisplayStyle?`        | Стиль отображения: struct, class, enum, tuple и т.д.     | `mirror.displayStyle == .struct`        |
| `superclassMirror` | `Mirror?`              | Mirror суперкласса (если есть)                           | рекурсивный обход иерархии              |
| `descendant(_:)`   | `Any?`                 | Доступ к вложенному свойству по пути                     | `mirror.descendant("address", "city")`  |
| `label` (в Child)  | `String?`              | Имя свойства (может быть [[nil]] для [[tuple]] без имён) | `label ?? "<unnamed>"`                  |
| `value` (в Child)  | `Any`                  | Значение свойства                                        | `print(value)`                          |

### 3. Основные DisplayStyle (что может вернуть mirror.displayStyle)

| DisplayStyle       | Для какого типа данных                     | Пример типа |
|--------------------|--------------------------------------------|-------------|
| `.struct`          | Структура                                  | `struct Point` |
| `.class`           | Класс (включая NSObject)                   | `class Person` |
| `.enum`            | Перечисление                               | `enum State` |
| `.tuple`           | Кортеж                                     | `(Int, String)` |
| `.optional`        | Опционал                                   | `String?` |
| `.collection`      | Коллекция (Array, Set, Dictionary)         | `[Int]` |
| `.dictionary`      | Словарь                                    | `[String: Int]` |
| `.set`             | Множество                                  | `Set<String>` |
| `.objC`            | Объект Objective-C                         | `NSString` |

### 4. Полные примеры кода (от простого к продвинутому)

#### Пример 1 — Базовый обход свойств

```swift
struct Person {
    let name: String
    var age: Int
    private let password: String = "secret"
}

let john = Person(name: "John", age: 30)
let mirror = Mirror(reflecting: john)

print("Тип:", mirror.subjectType)          // Person
print("Стиль:", mirror.displayStyle ?? "—") // struct

for child in mirror.children {
    let label = child.label ?? "<без имени>"
    print("\(label): \(child.value)")
}
// name: John
// age: 30
// password: secret   ← private тоже видно!
```

#### Пример 2 — Рекурсивный обход иерархии (суперклассы)

```swift
class Animal {
    var type = "Animal"
}

class Dog: Animal {
    var breed = "Labrador"
    var name = "Rex"
}

let rex = Dog()
var mirror = Mirror(reflecting: rex)

while let current = mirror {
    print("--- \(current.subjectType) ---")
    for child in current.children {
        print("  \(child.label ?? "?"): \(child.value)")
    }
    mirror = current.superclassMirror
}
// --- Dog ---
//   breed: Labrador
//   name: Rex
// --- Animal ---
//   type: Animal
```

#### Пример 3 — Mirror + [[enum]] с associated values

```swift
enum Message {
    case text(String)
    case image(URL, width: Int, height: Int)
    case typing
}

let msg = Message.image(URL(string: "https://example.com/cat.jpg")!, width: 300, height: 200)

let mirror = Mirror(reflecting: msg)

print("Enum case:", mirror.children.first?.label ?? "?")  // image

for child in mirror.children {
    print("Child:", child.value)
}
// Child: https://example.com/cat.jpg
// Child: 300
// Child: 200
```

#### Пример 4 — Mirror + [[generic]] функция (автоматический дамп объекта)

```swift
func dumpObject<T>(_ object: T, indent: Int = 0) {
    let mirror = Mirror(reflecting: object)
    let prefix = String(repeating: "  ", count: indent)
    
    print("\(prefix)Type: \(mirror.subjectType)")
    if let style = mirror.displayStyle {
        print("\(prefix)Style: \(style)")
    }
    
    for child in mirror.children {
        let label = child.label ?? "<unnamed>"
        print("\(prefix)- \(label): \(child.value)")
        
        // Рекурсивно для вложенных структур
        if Mirror(reflecting: child.value).children.count > 0 {
            dumpObject(child.value, indent: indent + 1)
        }
    }
    
    if let superMirror = mirror.superclassMirror {
        print("\(prefix)Superclass:")
        dumpObject(superMirror.subjectType, indent: indent + 1)
    }
}

struct Address {
    var city: String
    var street: String
}

struct User {
    var name: String
    var age: Int
    var address: Address
}

let user = User(name: "Anna", age: 28, address: Address(city: "Moscow", street: "Lenina 10"))
dumpObject(user)
```

Вывод будет примерно таким:

```
Type: User
Style: struct
- name: Anna
- age: 28
- address: Address(city: "Moscow", street: "Lenina 10")
  Type: Address
  Style: struct
  - city: Moscow
  - street: Lenina 10
```

### 5. Ограничения Mirror (чего нельзя сделать)

- Нельзя **изменять** значения свойств через Mirror  
- Нельзя вызвать **методы** объекта через Mirror  
- Нельзя получить **приватные свойства** enum с associated values (только публичные)  
- Нельзя получить **методы** и **инициализаторы** (только свойства и children)  
- Для **generic-типов** Mirror часто даёт мало информации  
- Не работает с **raw representable** enum без associated values

### 6. Лучшие практики и сценарии использования в 2026

- **Отладка** — самый частый случай (dumpObject выше)  
- **Логирование сложных объектов** — особенно в Sentry / Crashlytics  
- **Автогенерация UI-форм** — для дебаггера свойств  
- **Сравнение объектов** (diff) — полезно в тестах  
- **Сериализация** — когда [[Codable]] не подходит (например, динамические типы)  
- **Инспекторы в дебаггере** — [[Xcode]] уже использует Mirror под капотом

**Рекомендации 2026**:
- Не используйте Mirror в **продакшен-коде** (медленно и нестабильно)  
- Только для **debug**-сборок, инструментов и логирования  
- Для сериализации — предпочитайте **Codable**  
- Для производительности — избегайте рефлексии в горячих путях  
- В Swift 6 — Mirror полностью совместим со strict concurrency (но всё равно медленно)

**Короткий девиз 2026**:
> «Mirror — это швейцарский нож для отладки и анализа объектов в runtime.  
> Он не для продакшена, но незаменим в дебаггере, тестах и инструментах разработки.»
