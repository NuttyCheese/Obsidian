**`guard let`** — это способ **безопасного извлечения [[Optional]]**.

- Используется для **раннего выхода из функции**, если значение [[nil]]
    
- Работает с **Optional**
    
- Обеспечивает **сразу доступ к извлечённой переменной вне блока guard**
    
- Всегда требует **`else`** с `return`, `throw`, `break`, `continue` или `fatalError()`
    

> Проще говоря: `guard let` = «проверяем, есть ли значение, если нет — выходим, если есть — используем дальше».

---

## 2. Основные термины

| Термин              | Описание                                                                     |
| ------------------- | ---------------------------------------------------------------------------- |
| **[[Optional]]**    | Тип, который может быть `nil` (`Int?`, `String?`)                            |
| **[[Unwrapping]]**  | Извлечение значения из Optional                                              |
| **Guard Statement** | Условие, которое проверяет выражение и требует else блок                     |
| **Early Exit**      | Немедленный выход из функции при невыполнении условия                        |
| **Scope**           | Область видимости: переменная, извлечённая через guard, доступна после блока |

---

## 3. Основной синтаксис

```swift
func greet(name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    print("Hello, \(name)")
}

greet(name: "Alice") // Hello, Alice
greet(name: nil)     // No name provided
```

- `guard let` проверяет Optional
    
- Если `nil` → выполняется `else` и выход из функции
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
func printAge(_ age: Int?) {
    guard let age = age else {
        print("Age is missing")
        return
    }
    print("Age is \(age)")
}

printAge(25) // Age is 25
printAge(nil) // Age is missing
```

---

### Пример 2. Несколько Optional

```swift
func login(user: String?, password: String?) {
    guard let user = user, let password = password else {
        print("Missing credentials")
        return
    }
    print("Logging in \(user)")
}

login(user: "Alice", password: "123") // Logging in Alice
login(user: nil, password: "123")     // Missing credentials
```

- Можно извлекать **несколько Optional одновременно**
    

---

### Пример 3. Guard + условие

```swift
func checkNumber(_ num: Int?) {
    guard let num = num, num > 0 else {
        print("Invalid number")
        return
    }
    print("Valid number: \(num)")
}

checkNumber(10) // Valid number: 10
checkNumber(-5) // Invalid number
checkNumber(nil) // Invalid number
```

- Guard позволяет проверять **условия после извлечения**
    

---

### Пример 4. Guard в функциях с throw

```swift
enum LoginError: Error { case invalidCredentials }

func login(user: String?, password: String?) throws {
    guard let user = user, let password = password else {
        throw LoginError.invalidCredentials
    }
    print("Logged in \(user)")
}

try? login(user: "Alice", password: "123") // Logged in Alice
try? login(user: nil, password: "123")     // nil, error thrown
```

- Guard может использоваться с **throw** вместо return
    

---

### Пример 5. Guard в циклах

```swift
let numbers: [Int?] = [1, nil, 3, nil, 5]

for num in numbers {
    guard let num = num else { continue }
    print(num)
}

// Output:
// 1
// 3
// 5
```

- Guard можно использовать для **пропуска nil значений** внутри цикла
    

---

## 5. Особенности `guard let`

1. Используется для **раннего выхода** (`early exit`)
    
2. Извлечённая переменная доступна **после блока guard**
    
3. Требует **обязательный `else`**
    
4. Работает с **несколькими Optional и условиями**
    
5. Часто применяется для **входных проверок в функциях**
    

---

## 6. Итог

- **`guard let`** = безопасное извлечение Optional с немедленным выходом при nil
    
- Позволяет **уменьшить вложенность if-let**
    
- Улучшает читаемость и безопасность кода
    

---
