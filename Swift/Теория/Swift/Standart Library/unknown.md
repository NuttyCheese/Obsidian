## 1. Что такое `unknown`

В [[Swift]] **`unknown` напрямую не является типом языка**, но в современных версиях Swift (начиная с Swift 5.7) он используется в **сочетании с протоколами `any` и type casting**, чтобы обозначить **тип, который неизвестен на момент компиляции**.

Чаще всего встречается в следующих случаях:

1. **[[any]]** + `unknown` (например, `any SomeProtocol`) — указание, что конкретная реализация неизвестна
    
2. **Type casting через `as?`** — результат может быть неизвестного типа
    
3. В контексте **ошибок**: `catch (let error as any` [[Error]]) → `error` может быть `unknown`
    

> Проще говоря: `unknown` = «тип известен только во время выполнения, компилятору заранее неизвестен».

---

## 2. Основные термины

| Термин                   | Описание                                                                       |
| ------------------------ | ------------------------------------------------------------------------------ |
| **Any**                  | Любой тип ([[Value Type]] или reference type)                                  |
| **[[AnyObject]]**        | Любой класс ([[Reference Type]])                                               |
| **unknown type**         | Тип, который неизвестен на этапе компиляции                                    |
| **Type casting (`as?`)** | Попытка привести тип к конкретному типу                                        |
| **Existential type**     | Тип, который может содержать экземпляр любого типа, соответствующего протоколу |

---

## 3. Примеры от простого к сложному

### Пример 1. Any и unknown

```swift
let value: any Any = 42
// Swift не знает конкретный тип value на этапе компиляции
```

- `value` может быть любым типом
    
- Чтобы использовать конкретные свойства → нужен **type cast**
    

---

### Пример 2. Type casting

```swift
let value: Any = "Hello"
if let stringValue = value as? String {
    print(stringValue) // Hello
}
```

- Без `as?` компилятор не знает тип → `unknown`
    
- Приведение типа безопасное, возвращает Optional
    

---

### Пример 3. Работа с протоколами

```swift
protocol Greetable {
    func greet()
}

struct Person: Greetable {
    func greet() { print("Hello") }
}

let someone: any Greetable = Person() // someone имеет unknown тип на этапе компиляции
someone.greet()
```

- `any Greetable` — existential type, конкретная реализация неизвестна
    

---

### Пример 4. Ошибки и unknown

```swift
enum MyError: Error { case failed }

func risky() throws {
    throw MyError.failed
}

do {
    try risky()
} catch (let error) {
    print(type(of: error)) // Unknown type of error at compile time
}
```

- `error` имеет **неизвестный конкретный тип**
    
- Можно привести к конкретному через `as? MyError`
    

---

### Пример 5. Any + unknown + функции

```swift
func printAny(_ value: any Any) {
    switch value {
    case let int as Int:
        print("Int:", int)
    case let string as String:
        print("String:", string)
    default:
        print("Unknown type")
    }
}

printAny(42)        // Int: 42
printAny("Hello")   // String: Hello
printAny([1,2,3])   // Unknown type
```

- При неизвестном типе используется **default case**
    

---

## 4. Особенности unknown

1. **Неизвестный тип** → компилятор не знает на этапе компиляции
    
2. Обычно используется с **`any, Any, AnyObject, existential types`**
    
3. Для работы с конкретным типом требуется **type casting**
    
4. Позволяет писать **универсальный и безопасный код** для любых типов
    

---

## 5. Итог

- **unknown** = тип, известный только во время выполнения
    
- Используется с **Any, any [[Protocol]], ошибки, type casting**
    
- Позволяет **универсально обрабатывать разные типы и ошибки**
    

---
