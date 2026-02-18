**Bool** — это самый простой и один из самых часто используемых типов в Swift. Он может принимать **только два значения**: `true` и `false`.

Это фундаментальный строительный блок для:
- всех условий (`if`, `guard`, `switch`, `while`, `for`)
- логических выражений
- toggle-состояний (включено/выключено)
- флагов (доступно/запрещено, успешно/ошибка)
- свойств типа «isLoading», «isDarkMode», «hasError» и т.д.

В 2026 году `Bool` остаётся неизменным, но его использование стало ещё более строгим и выразительным благодаря Swift 6 strict concurrency и новым паттернам.

### 1. Основные факты о Bool (2026 актуально)

| Характеристика                          | Значение / Особенность                                      | Важные детали |
|-----------------------------------------|-------------------------------------------------------------|---------------|
| Возможные значения                      | `true` или `false`                                          | Только два, никаких `nil`, `1/0`, `YES/NO` |
| Размер в памяти                         | 1 байт                                                      | Очень компактный |
| Value Type                              | Да — копируется при присваивании                            | Нет retain cycle |
| Default значение (при инициализации)    | Нет значения по умолчанию — обязательно инициализировать   | `var flag: Bool` → ошибка компиляции |
| Поддержка в SwiftUI / Combine           | Полная — `@State var isOn: Bool`, `Publisher<Bool, Never>` | Самый частый тип для toggle |
| Логические операторы                   | `&&` (AND), `||` (OR), `!` (NOT), `^` (XOR)                | Все работают как ожидается |
| Сравнение                               | `==`, `!=`                                                  | `true == true`, `false != true` |
| Конвертация из других типов             | Только явно: `Bool(1)`, `Bool("true")` — не рекомендуется  | Лучше писать `value != 0` или `text.lowercased() == "true"` |

### 2. Самые частые паттерны использования Bool в 2026 году

#### Паттерн 1 — Флаги состояния (самый популярный)

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var hasError = false
    @Published var isDarkMode = false
    
    func fetchData() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            // загрузка
        } catch {
            hasError = true
        }
    }
}
```

#### Паттерн 2 — Toggle с анимацией (SwiftUI + UIKit)

```swift
// SwiftUI
Toggle("Тёмная тема", isOn: $isDarkMode)
    .animation(.spring(), value: isDarkMode)

// UIKit (с UIButton.Configuration)
var config = button.configuration ?? .filled()
config.title = isDarkMode ? "Светлая тема" : "Тёмная тема"
button.configuration = config
```

#### Паттерн 3 — Bool как результат сложного условия

```swift
let canProceed = 
    user.isAuthenticated &&
    !cart.isEmpty &&
    paymentMethod != nil &&
    Date.now < orderDeadline

if canProceed {
    placeOrder()
} else {
    showValidationAlert()
}
```

#### Паттерн 4 — Bool + optional chaining (очень популярно)

```swift
if user?.isPremium == true {
    showPremiumFeatures()
}

// или более безопасно
if let isPremium = user?.isPremium, isPremium {
    // ...
}
```

#### Паттерн 5 — Bool в switch / if case (с associated values)

```swift
enum AuthState {
    case signedIn(user: User)
    case signedOut
}

let state: AuthState = .signedIn(user: currentUser)

if case .signedIn = state {
    print("Авторизован")
}
```

### 3. Полезные расширения Bool (очень популярны в 2026)

```swift
extension Bool {
    /// Инвертировать значение
    mutating func toggle() {
        self = !self
    }
    
    /// Выполнить замыкание только если true
    func ifTrue(_ closure: () -> Void) {
        if self { closure() }
    }
    
    /// Выполнить замыкание только если false
    func ifFalse(_ closure: () -> Void) {
        if !self { closure() }
    }
    
    /// Вернуть значение или дефолт
    func or(_ defaultValue: Bool) -> Bool {
        self || defaultValue
    }
}

// Использование
var isDarkMode = false
isDarkMode.toggle()           // true
isDarkMode.ifTrue { applyDarkTheme() }
```

### 4. Лучшие практики Bool в Swift 2026

- **Называйте свойства с префиксом is / has / can / should**  
  `isLoading`, `hasError`, `canProceed`, `shouldShowOnboarding` — стандарт  
- **Избегайте двойного отрицания** — `!isNotLoggedIn` → лучше `isLoggedIn`  
- **Используйте Bool вместо Int для флагов** — `status == 1` → `isActive`  
- **Не храните Bool в Optional**, если это не нужно — `Bool?` редко оправдано  
- **В SwiftUI** — `@State`, `@Binding`, `@Published` для Bool — это основа toggle, sheet, alert  
- **В Combine / async** — `Publisher<Bool, Never>`, `AsyncStream<Bool>`  
- **Swift 6 strict concurrency** — Bool полностью Sendable и безопасен  
- **Документируйте** — пиши комментарий «isPremium — флаг премиум-подписки пользователя»

**Короткий девиз 2026**:
> Bool — это **самый честный и быстрый** тип в Swift: только true или false, никаких полутонов.  
> В 2026 году используй `is`, `has`, `can`, `should` в именах, toggle() для инверсии, `ifTrue`/`ifFalse` для читаемости и `@MainActor` для UI-состояний.  
> Это **основа** всех условий, состояний и toggle в твоём приложении.

Удачи с чистыми и понятными логическими состояниями в коде! ✅