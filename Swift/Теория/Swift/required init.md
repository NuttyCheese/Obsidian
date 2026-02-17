### Что такое `required init`

`required init` — это специальный модификатор инициализатора в **классах**, который говорит компилятору:

> «Любой подкласс этого класса **обязан** реализовать (или унаследовать) этот инициализатор».

Ключевые особенности:

- Применяется **только к классам** (не к [[struct]] / [[enum]] / [[actor]])  
- Работает **только с designated initializers** (не convenience)  
- Если у суперкласса есть `required init`, то **каждый** подкласс **должен** либо:
  - реализовать этот [[init]] самостоятельно  
  - унаследовать его (если суперкласс предоставляет реализацию)  
- Если подкласс **не реализует** required init и не наследует его → **ошибка компиляции**

### Когда и зачем используют `required init`

| Сценарий                                      | Почему нужен именно `required` | Пример |
|-----------------------------------------------|--------------------------------|--------|
| Базовый класс с обязательным параметром для всех подклассов | Подклассы не смогут создать экземпляр без передачи этого параметра | `required init(identifier: String)` |
| Абстрактный / базовый класс с обязательной инициализацией | Заставляет подклассы правильно настроить объект | `required init(viewModel: ViewModel)` |
| Протокол с init-требованием + классы | Обязывает все классы, реализующие протокол, иметь этот init | `required init(from decoder: Decoder)` в `Codable` |
| Фреймворки / библиотеки (UI-компоненты, ViewModel, Controller) | Гарантирует, что подклассы будут правильно инициализированы | `required init(nibName: String?, bundle: Bundle?)` |
| NSCoding / NSSecureCoding (архивация)         | Обязательный `init(coder:)` для всех подклассов | `required init?(coder: NSCoder)` |

### Примеры required init

#### Пример 1 — Обязательный параметр для всех подклассов

```swift
open class BaseViewController: UIViewController {
    
    let viewModel: ViewModel
    
    // required — все подклассы обязаны вызвать этот init
    public required init(viewModel: ViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    // Обязательно реализуем init(coder:) из NSCoding
    public required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

class ProfileViewController: BaseViewController {
    // Компилятор заставит вызвать required init суперкласса
    required init(viewModel: ViewModel) {
        super.init(viewModel: viewModel)
        
        // дополнительная настройка
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

#### Пример 2 — required init в протоколе + класс

```swift
protocol Identifiable {
    init(identifier: String)
}

open class BaseEntity: Identifiable {
    let identifier: String
    
    public required init(identifier: String) {
        self.identifier = identifier
    }
}

class User: BaseEntity {
    var name: String = ""
    
    // Компилятор заставит реализовать init(identifier:)
    required init(identifier: String) {
        super.init(identifier: identifier)
    }
}
```

#### Пример 3 — required init(coder:) — самый частый случай

```swift
open class BaseView: UIView {
    
    public required init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    // Обязательно для всех подклассов
    public required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
    
    private func setup() {
        // общая настройка
    }
}

class CustomButton: BaseView {
    // Компилятор требует init?(coder:)
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        // дополнительная настройка кнопки
    }
}
```

### Лучшие практики required init в Swift 2026

- **Используй required**, когда:
  - класс предназначен для наследования (`open class`)  
  - init содержит **обязательную настройку** (DI, setup, [[KVO]] и т.д.)  
  - нужно гарантировать вызов [[super.init]] в подклассах  
  - класс реализует `NSCoding` / `NSSecureCoding` (требуется `required init?(coder:)`)

- **Не используй required**, если:
  - класс `final` (наследовать нельзя)  
  - [[init]] — convenience (не designated)  
  - класс не предназначен для наследования (`public final class`)

- **Swift 6 strict concurrency** — required init должен быть `@MainActor` или [[Sendable]]-safe, если класс используется в UI/Concurrency  
- **Документируйте** — пиши комментарий «required init — обязательный для всех подклассов»

**Короткий девиз 2026**:
> «required init — это когда ты говоришь всем подклассам: «ты обязан вызвать этот инициализатор, иначе не скомпилируешься».  
> В 2026 году это **обязательный** инструмент для всех open-классов, которые должны правильно инициализироваться в подклассах.»

