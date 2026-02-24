**@autoclosure** — это атрибут в [[Swift]], который позволяет автоматически **оборачивать переданное выражение в замыкание** без явных фигурных скобок `{}`. Это делает код чище, естественнее и позволяет **откладывать вычисление** выражения до момента, когда оно действительно понадобится.

### Ключевые характеристики @autoclosure (актуально на 2026 год)

| Характеристика                                   | Описание                                                                              | Важные нюансы 2026                                         |
| ------------------------------------------------ | ------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Тип параметра                                    | `@autoclosure () -> T` (или `@autoclosure @escaping () -> T`)                         | Возвращаемый тип — любой ([Bool], [[String]], Void и т.д.) |
| Вычисление                                       | **Ленивое** — выражение не выполняется, пока не вызовут замыкание                     | Экономит ресурсы при дорогих вычислениях                   |
| Обязательно одно выражение                       | Нельзя передать несколько выражений или блок кода                                     | Только одно выражение                                      |
| Не escaping по умолчанию                         | Без `@escaping` замыкание не может быть сохранено надолго (нельзя хранить в свойстве) | Часто добавляют `@escaping`                                |
| Нельзя использовать с inout-параметрами          | `@autoclosure` + [[inout]] — запрещено компилятором                                   | —                                                          |
| Работает только с **неименованными** параметрами | Нельзя написать `@autoclosure let condition: () -> Bool`                              | Только без имени                                           |
| Можно комбинировать с `@escaping`                | `@autoclosure @escaping () -> Bool` — замыкание можно сохранить и вызвать позже       | Очень популярно для отложенных проверок                    |

### Почему @autoclosure так любят (преимущества)

1. Код выглядит **натурально**, как обычное условие

```swift
// Без @autoclosure
logIfTrue({ expensiveCheck() })           // некрасиво

// С @autoclosure
logIfTrue(expensiveCheck())               // красиво и естественно
```

2. **Отложенное вычисление** — экономия ресурсов

```swift
// expensiveCheck() НЕ вызывается, если условие уже ложно
guard isUserLoggedIn() else { return }
guard expensiveNetworkCheck() else { return }  // не выполнится, если первый guard сработал
```

3. Поддерживает **short-circuit evaluation** (ленивые && и ||)

```swift
func require(_ condition: @autoclosure () -> Bool, _ message: String) {
    assert(condition(), message)
}

require(user.isAdmin && expensiveAdminCheck(), "Доступ запрещён")
// expensiveAdminCheck() НЕ вызовется, если user.isAdmin == false
```

### Самые популярные и реальные сценарии использования в 2026 году

#### 1. Замена assert / precondition / guard с ленивым вычислением

```swift
func require(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = "Requirement failed") {
    assert(condition(), message())
}

require(user.balance >= amount, "Недостаточно средств: \(user.balance) < \(amount)")
// message вычисляется только при краше
```

#### 2. Логирование с условием (очень популярно)

```swift
func logIfEnabled(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String) {
    guard isLoggingEnabled && condition() else { return }
    print("[DEBUG] \(message())")
}

logIfEnabled(user.isPremium) {
    "Пользователь \(user.id) имеет премиум-аккаунт"
}
// message вычисляется только если логи включены И пользователь премиум
```

#### 3. Отложенная проверка в замыканиях (с @escaping)

```swift
class FormValidator {
    private var validations: [() -> Bool] = []
    
    func addValidation(_ condition: @autoclosure @escaping () -> Bool) {
        validations.append(condition)
    }
    
    func isValid() -> Bool {
        validations.allSatisfy { $0() }
    }
}

let validator = FormValidator()
validator.addValidation(email.isValidFormat)
validator.addValidation(password.count >= 8)
validator.addValidation(expensiveServerSideCheck())  // НЕ вызывается сразу
```

#### 4. Условное выполнение дорогих операций

```swift
func executeIf(_ condition: @autoclosure () -> Bool, _ action: () -> Void) {
    if condition() {
        action()
    }
}

executeIf(user.isAdmin && server.isReachable()) {
    performAdminAction()  // не выполнится, если хотя бы одно условие ложно
}
```

### Когда НЕ использовать @autoclosure (ловушки 2026)

1. **Когда выражение имеет побочные эффекты** и должно выполняться всегда

```swift
// Ошибка
logIfTrue(incrementCounter())  // счётчик НЕ увеличится, если условие ложно!

// Правильно
incrementCounter()
logIfTrue(true)
```

2. **Когда нужно передать несколько выражений**

```swift
// Нельзя
logIfTrue(user.isLoggedIn && user.hasPremium)  // Ошибка компиляции
```

3. **Когда замыкание должно храниться надолго без @escaping**

```swift
// Ошибка — компилятор не даст
var storedCondition: () -> Bool = { true && expensive() }  // без @autoclosure
```

### Лучшие практики @autoclosure в Swift 2026

- **Используйте** для **ленивых условий**, **assert**, **логов**, **валидаций**, **guard**-подобных функций  
- **Всегда добавляйте `@escaping`**, если замыкание сохраняется (в массиве, свойстве и т.д.)  
- **Комбинируйте** с `@autoclosure` для сообщения об ошибке — это стандарт в assert/precondition  
- **Не используйте** для выражений с **побочными эффектами** (инкремент, сетевые вызовы)  
- **В SwiftUI** — `@autoclosure` почти не нужен (используйте `@ViewBuilder`, computed properties)  
- **Документируйте** — пишите комментарий:

```swift
/// Проверяет условие лениво и выводит сообщение только при провале
func require(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = "Assertion failed")
```

**Короткий итог 2026**:
> `@autoclosure` — это **автоматическое оборачивание выражения в замыкание**, которое позволяет писать естественный код и откладывать вычисления.  
> В 2026 году:  
> - самый популярный кейс — условия в `assert`, `guard`, логи, валидация форм  
> - часто комбинируется с `@escaping` для хранения замыкания  
> - делает код **красивее** и **эффективнее** за счёт ленивого выполнения  
> - это **один из самых любимых** инструментов Swift-разработчиков для чистоты кода
