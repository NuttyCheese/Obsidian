**super.init** — это вызов **designated инициализатора** (обязательного конструктора) суперкласса из подкласса.

В [[Swift]] это **единственный правильный способ** инициализировать наследуемый класс, и он имеет строгие правила, которые компилятор проверяет очень жёстко (особенно в Swift 6 с полной проверкой concurrency).

### Когда и зачем вызывать super.init

| Ситуация                                                                                 | Нужно ли вызывать super.init?             | Где именно вызывать                | Примечание / ошибка, если забыть                                                            |
| ---------------------------------------------------------------------------------------- | ----------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------- |
| Подкласс наследует от **класса** ([[UIViewController]], [[UIView]], [[NSObject]] и т.д.) | **Обязательно**                           | В каждом designated init подкласса | Ошибка компиляции: 'super.init' isn't called on all paths before returning from initializer |
| Подкласс реализует протокол с **[[required init]]**                                      | **Обязательно** (если суперкласс требует) | В каждом required init подкласса   | Ошибка: required initializer must call super.init                                           |
| Подкласс имеет **convenience init**                                                      | **Нет** (но можно)                        | —                                  | convenience init должен вызывать другой init этого же класса                                |
| Подкласс — **[[final]] [[class]]**                                                       | **Да** (если наследует от другого класса) | В designated init                  | final не отменяет необходимость вызова super.init                                           |
| [[SwiftUI]] View / ObservableObject                                                      | **Нет** (они не наследуют от NSObject)    | —                                  | SwiftUI использует struct + @Observable                                                     |

### Правила вызова super.init (Swift 6+)

1. **super.init должен быть вызван до использования self**  
   → Нельзя обращаться к свойствам [[self]] / self.someMethod() до вызова super.init

2. **Все свойства подкласса должны быть инициализированы до super.init**  
   → Или иметь [[default]]-значение, или быть инициализированы до вызова

3. **super.init вызывается только в designated init**  
   → convenience init вызывает другой init этого же класса

4. **Если суперкласс имеет required init** — подкласс обязан его реализовать и вызвать super

### Примеры правильного и неправильного использования

#### Правильно (классический [[UIViewController]])

```swift
class BaseViewController: UIViewController {
    let viewModel: ViewModel
    
    init(viewModel: ViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)  // правильно: после инициализации свойств
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

class ProfileViewController: BaseViewController {
    let profileImageView = UIImageView()
    
    init(viewModel: ProfileViewModel) {
        super.init(viewModel: viewModel)  // правильно: передаём в super
        
        // после super можно использовать self
        profileImageView.contentMode = .scaleAspectFill
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

#### Неправильно (ошибки компиляции)

```swift
class WrongVC: UIViewController {
    let titleLabel = UILabel()
    
    init() {
        titleLabel.text = "Hello"           // Ошибка! self до super.init
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)            // Ошибка! titleLabel не инициализирован
    }
}
```

### Лучшие практики super.init в Swift 2026

- **Инициализируй все свойства подкласса до super.init**  
- **Вызывай super.init как можно раньше** (но после инициализации своих свойств)  
- **required init?(coder:)** — всегда реализуй, даже если fatalError  
- **@MainActor** — если класс UI-related — пометь весь класс или init как @MainActor  
- **Swift 6 strict concurrency** — super.init должен быть в правильном контексте изоляции  
- **Документируйте** — пиши комментарий «super.init после инициализации всех свойств»

**Короткий девиз 2026**:
> «super.init — это когда ты говоришь: «я закончил настраивать себя, теперь пусть суперкласс настроит своё».  
> Вызывай его **после** инициализации всех своих свойств и **до** использования self.  
> Забыл — компилятор напомнит жёстко.»
