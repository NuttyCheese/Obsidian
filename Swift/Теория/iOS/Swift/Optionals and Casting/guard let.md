**`guard let`** — это конструкция в [[Swift]], которая **безопасно разворачивает [[Optional]]** и **немедленно выходит** из текущей области видимости (функции, метода, цикла, замыкания), если значение оказалось [[nil]].

Это один из самых любимых и рекомендуемых паттернов в современном Swift (2025–2026), потому что он:

- резко уменьшает вложенность кода  
- делает логику линейной и читаемой  
- заставляет обрабатывать `nil`-случай **сразу**, а не откладывать  

> Проще говоря:  
> «Если значение есть — идём дальше с развёрнутым значением.  
> Если нет — выходим прямо сейчас и делаем что-то в else».

### 1. Почему `guard let` лучше, чем `if let` (главные преимущества)

| Ситуация / Проблема              | [[if let]] (старый стиль)                       | `guard let` (современный стиль)           | Почему `guard` выигрывает    |
| -------------------------------- | ----------------------------------------------- | ----------------------------------------- | ---------------------------- |
| Много проверок в начале функции  | Глубокая вложенность if-ов                      | Все проверки на одном уровне              | Код линейный, читаемость ×10 |
| Ранний возврат при ошибке        | Приходится писать else с [[return]] в каждом if | Один else на все guard-ы                  | Меньше дублирования          |
| Длинная функция с валидацией     | Логика «сдвигается вправо»                      | Логика остаётся на левом краю             | Легче читать и поддерживать  |
| Обязательность обработки [[nil]] | Можно забыть else                               | `else` обязателен (компилятор заставляет) | Меньше багов                 |

### 2. Полный синтаксис и все варианты (2026 стандарт)

#### Вариант 1 — Классический guard let

```swift
func greet(name: String?) {
    guard let name else {
        print("Имя не указано")
        return
    }
    
    print("Привет, \(name)!")
}
```

#### Вариант 2 — Несколько guard let подряд (самый популярный паттерн)

```swift
func login(user: String?, password: String?, deviceToken: String?) throws {
    guard let user else {
        throw AuthError.missingCredentials("user")
    }
    guard let password else {
        throw AuthError.missingCredentials("password")
    }
    guard let deviceToken else {
        throw AuthError.missingDeviceToken
    }
    
    // здесь уже все значения точно не nil
    print("Логин:", user, "Пароль:", password, "Токен:", deviceToken)
}
```

#### Вариант 3 — guard let + условие (очень мощно)

```swift
func processAge(_ age: Int?) {
    guard let age, age >= 18 else {
        print("Доступ запрещён: возраст < 18 или не указан")
        return
    }
    
    print("Добро пожаловать! Возраст:", age)
}
```

#### Вариант 4 — guard let в цикле (часто используется с коллекциями)

```swift
let optionalNumbers: [Int?] = [1, nil, 3, nil, 5, 7]

for number in optionalNumbers {
    guard let number else { continue }
    
    if number % 2 == 0 {
        print("Чётное:", number)
        continue
    }
    
    print("Нечётное:", number)
}
```

#### Вариант 5 — guard в [[async]]-функциях (самый частый сценарий 2026)

```swift
func fetchUserProfile(id: String?) async throws -> User {
    guard let id else {
        throw AuthError.missingUserID
    }
    
    let (data, _) = try await URLSession.shared.data(from: userURL(for: id))
    return try JSONDecoder().decode(User.self, from: data)
}
```

### 3. Лучшие практики guard let в Swift 2026

- **Пиши guard в начале функции** — все проверки и early exit идут сразу  
- **Используй guard для всех входных [[Optional]]** — это резко снижает вложенность  
- **Комбинируй** несколько условий в одном guard:

  ```swift
  guard let user = user,
        let email = user.email,
        !email.isEmpty,
        email.contains("@") else {
      throw ValidationError.invalidEmail
  }
  ```

- **В async** — guard + `throw` — золотой стандарт  
- **В циклах** — `guard ... else { continue }` или `break`  
- **Swift 6 strict concurrency** — guard полностью безопасен, но код после guard должен учитывать актор  
- **Документируйте** — пиши комментарий «guard let — проверка и извлечение userID»

**Короткий девиз 2026**:
> `guard let` — это «если что-то не так — выходим сразу, иначе идём дальше с развёрнутым значением».  
> В 2026 году используй его **везде**, где есть Optional:  
> - в начале каждой функции  
> - перед использованием Optional  
> - в async-коде с throws  
> Это **самый читаемый** и **самый безопасный** способ работы с Optional.
