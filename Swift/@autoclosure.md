**Определение:**  
`@autoclosure` — это атрибут в [[Swift]], который автоматически **оборачивает выражение в замыкание ([[closure]]) без скобок**. Это позволяет писать более читаемый код и откладывать вычисление значения до момента использования.

Иными словами: **вы передаёте выражение, а Swift превращает его в замыкание, которое выполнится только когда нужно**.

---

## Основные моменты

|Характеристика|Описание|
|---|---|
|Тип|Атрибут функции параметра|
|Применение|Часто используется для условий, логов, assert, lazy evaluation|
|Вычисление|Ленивая (отложенная), значение не вычисляется до вызова closure|
|Синтаксис|`func example(param: @autoclosure () -> Bool)`|

---

## Простая демонстрация

```swift
func logIfTrue(_ condition: @autoclosure () -> Bool) {
    if condition() {
        print("Условие истинно")
    } else {
        print("Условие ложно")
    }
}

logIfTrue(2 > 1)
```

**Что происходит:**

1. `2 > 1` — обычное выражение
    
2. Благодаря `@autoclosure` Swift превращает его в `{ 2 > 1 }`
    
3. Выражение вычисляется только при вызове `condition()`
    

**Вывод:**

```
Условие истинно
```

---

## Почему это удобно

Без `@autoclosure` пришлось бы писать так:

```swift
func logIfTrue(_ condition: () -> Bool) {
    if condition() {
        print("Условие истинно")
    }
}

logIfTrue({ 2 > 1 }) // <--- нужно явно писать closure
```

`@autoclosure` делает вызов чище:

```swift
logIfTrue(2 > 1) // без {}
```

---

## Пример с **побочными эффектами**

```swift
func checkAndPrint(_ condition: @autoclosure () -> Bool) {
    print("Начинаем проверку")
    if condition() {
        print("Условие выполнено")
    } else {
        print("Условие не выполнено")
    }
    print("Проверка завершена")
}

checkAndPrint(expensiveCalculation())

func expensiveCalculation() -> Bool {
    print("Выполняется дорогая операция")
    return true
}
```

**Вывод:**

```
Начинаем проверку
Выполняется дорогая операция
Условие выполнено
Проверка завершена
```

- `expensiveCalculation()` вызывается **только когда `condition()` вызывается**.
    
- Можно совмещать с **short-circuit evaluation**.
    

---

## Использование с AND/OR и short-circuit

`@autoclosure` часто встречается в функциях вроде `assert`:

```swift
func myAssert(_ condition: @autoclosure () -> Bool, _ message: String) {
    if !condition() {
        print("Assertion failed: \(message)")
    }
}

let x = 5
myAssert(x > 10, "x должно быть больше 10")
```

- `x > 10` **не вычисляется до вызова closure**
    
- Удобно для сложных проверок с дорогими вычислениями или сетевыми условиями.
    

---

## Расширенный пример с UIKit

Представим **валидацию формы**:

```swift
func validateField(_ field: UITextField, _ condition: @autoclosure () -> Bool, message: String) -> Bool {
    if !condition() {
        print("Ошибка: \(message)")
        field.backgroundColor = .red
        return false
    }
    field.backgroundColor = .white
    return true
}

let nameField = UITextField()
nameField.text = ""

let isNameValid = validateField(nameField, nameField.text?.isEmpty == false, message: "Имя не может быть пустым")
```

**Пояснение:**

- `nameField.text?.isEmpty == false` автоматически оборачивается в замыкание
    
- Вычисляется **только внутри `validateField`**
    
- Можно легко добавить **несколько проверок подряд** с `&&` или `||`
    

---

## Пример с “отложенной логикой”

```swift
func executeIfTrue(_ condition: @autoclosure () -> Bool, action: () -> Void) {
    if condition() {
        action()
    }
}

executeIfTrue(2 > 3) {
    print("Никогда не выполнится")
}

executeIfTrue(3 > 2) {
    print("Выполнится") // <- Вывод
}
```

---

## Важные замечания

1. `@autoclosure` работает **только с одним выражением**. Нельзя передавать сразу несколько выражений.
    
2. Можно использовать вместе с `@escaping`:
    

```swift
func delayed(_ condition: @autoclosure @escaping () -> Bool, completion: () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        if condition() {
            completion()
        }
    }
}
```

- Это позволяет хранить **closure на будущее** (например, для UI или сетевых проверок).
    

---

## Итог

- `@autoclosure` делает код **короче и читаемее**
    
- Полезен для:
    
    - assert и precondition
        
    - lazy evaluation
        
    - UI-валидации форм
        
- Часто комбинируется с **short-circuit evaluation**
    
- Можно использовать вместе с `@escaping` для отложенного выполнения
    

---
