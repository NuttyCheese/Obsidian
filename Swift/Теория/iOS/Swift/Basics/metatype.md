**Metatype** в [[Swift]] — это **тип, который описывает сам тип** (а не экземпляр этого типа).  

По сути, metatype — это **тип самого типа**.

### Основные варианты metatype в Swift

| Запись                | Что это такое                                       | Тип выражения               | Когда используется чаще всего      |
| --------------------- | --------------------------------------------------- | --------------------------- | ---------------------------------- |
| `T.self`              | Metatype для конкретного типа `T`                   | `T.Type`                    | Самый распространённый вариант     |
| `T.Type`              | Сам тип [[metatype]] (как тип переменной/параметра) | Метатип (метатип типа)      | В [[generic]]s, параметрах функций |
| `(any Protocol).self` | Metatype для экзистенциального типа                 | `(`[[any Protocol]]`).Type` | Редко, в основном с `any`          |
| `Any.self`            | Metatype для [[Any]]                                | `Any.Type`                  | Очень редко                        |

### Самые важные и часто используемые паттерны (2025–2026)

#### 1. Получение metatype через .[[self]] (самый частый случай)

```swift
let type1 = String.self          // тип String.Type
let type2 = Int.self             // тип Int.Type
let type3 = (Int, String).self   // тип (Int, String).Type
let type4 = [Double].self        // тип Array<Double>.Type

print(type1)                     // String
```

#### 2. Передача типа как параметра (очень популярно в generics и фабриках)

```swift
func createInstance<T>(of type: T.Type) -> T {
    // здесь можно создавать экземпляры, если есть init()
    fatalError("Не реализовано")
}

let strType = String.self
let newString = createInstance(of: strType)  // String
```

#### 3. Проверка типа через metatype (вместо `is` / `as?`)

```swift
func printTypeInfo<T>(_ value: T, expected: T.Type) {
    if type(of: value) == expected {
        print("Тип совпадает: \(expected)")
    }
}

let value = 42
printTypeInfo(value, expected: Int.self)     // Тип совпадает: Int
```

#### 4. Metatype в протоколах и generics (самый мощный паттерн)

```swift
protocol Instantiable {
    static func create() -> Self
}

extension Instantiable {
    static func create() -> Self {
        Self() // требует, чтобы был init()
    }
}

struct User: Instantiable {
    let name: String = "Anonymous"
}

let userType: any Instantiable.Type = User.self
let user = userType.create()  // создаёт User
```

#### 5. Metatype + reflection (часто в тестах / сериализации)

```swift
let types: [any Any.Type] = [Int.self, String.self, User.self]

for type in types {
    print("Тип:", type)
    // Можно проверить conforms(to:), зеркалить и т.д.
}
```

### Когда metatype критически важен

- Создание экземпляров по типу (фабрики, dependency injection)
- Регистрация типов в системах (например, в [[SwiftUI]] ViewBuilder, в тестах)
- Сравнение типов (`===` для metatype работает)
- Универсальные коллекции типов (`[Any.Type]`, `[any Protocol.Type]`)
- Работа с generics, где нужен конкретный `.Type`
- Динамическая загрузка типов (редко, но встречается в плагинах/модулях)

### Лучшие практики работы с metatype в Swift 2026

- **Всегда** используйте `.self` для получения metatype (`String.self`, `MyStruct.self`)
- **Предпочитайте** конкретные типы (`User.self`) вместо `Any.Type`, если возможно
- **Не храните** большие коллекции `[Any.Type]` — это редко оправдано
- **В SwiftUI** — metatype часто используется неявно (ViewBuilder, `View.self`)
- **В Swift 6 strict concurrency** — metatype сам по себе [[Sendable]], если базовый тип `Sendable`
- **Документируйте** — пишите комментарий `User.self — metatype для создания экземпляров User`

**Короткий итог 2026**:
> Metatype (`T.Type`) — это **тип самого типа**, а `T.self` — способ его получить.  
> В 2026 году метатипы чаще всего нужны для:  
> - передачи типа как параметра  
> - фабрик и DI  
> - регистрации типов  
> - reflection и динамической работы  
> Это **не повседневный инструмент**, но **очень мощный**, когда нужно работать именно с типами, а не с их значениями.
