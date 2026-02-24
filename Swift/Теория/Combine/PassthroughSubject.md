**`PassthroughSubject`** — это один из самых простых и часто используемых **Publisher** в **Combine**.

Его ключевая особенность:  
- **не хранит** никакого значения (в отличие от `CurrentValueSubject`)  
- **передаёт** каждое новое значение **всем текущим подписчикам** сразу  
- **не имеет начального значения** — новые подписчики получают только те события, которые придут **после** их подписки  
- **идеален** для **событийного** потока (кнопки, уведомления, кастомные события, broadcast)

Простыми словами:  
`PassthroughSubject` — это как **радиостанция**:  
кто включил приёмник (подписался) — слышит всё, что идёт в эфир с этого момента.  
Кто подключился позже — пропускает предыдущие песни.

### Когда использовать PassthroughSubject (реальные кейсы 2025–2026)

| Сценарий                                      | Почему именно PassthroughSubject                             | Альтернатива |
|-----------------------------------------------|---------------------------------------------------------------|--------------|
| Кастомные события (нажатие кнопки, свайп, выбор в меню) | Нет начального состояния, только события                     | `Void` или `()` |
| Глобальный Event Bus / Notification заменитель | Легко рассылать события всем желающим                          | `NotificationCenter` (хуже типобезопасность) |
| Передача одноразовых команд (dismiss, showAlert, reload) | Не нужно хранить последнее состояние                           | `@Published` (если нужно состояние) |
| Имитация делегата или callback в реактивном стиле | Один `send`, все слушатели получают событие                    | `PassthroughSubject` + `sink` |
| Трансляция событий из одного слоя в другой (Coordinator → ViewModel) | Чистая передача без хранения прошлого                         | `CurrentValueSubject` (если нужно текущее состояние) |

### Самые популярные и рекомендуемые паттерны (2026)

#### 1. Простейший PassthroughSubject (событие без данных)

```swift
let buttonTap = PassthroughSubject<Void, Never>()

buttonTap
    .sink { _ in
        print("Кнопка нажата!")
    }
    .store(in: &cancellables)

// где-то в UI
buttonTap.send(())  // или .send(())
```

#### 2. Событие с данными (самый частый)

```swift
let itemSelected = PassthroughSubject<String, Never>()

itemSelected
    .sink { item in
        print("Выбран элемент:", item)
    }
    .store(in: &cancellables)

// в таблице или коллекции
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let item = items[indexPath.row]
    itemSelected.send(item)
}
```

#### 3. PassthroughSubject как замена делегату

```swift
protocol CustomViewDelegate: AnyObject {
    func didFinishLoading()
}

class CustomView {
    let didFinishLoading = PassthroughSubject<Void, Never>()
    
    func loadData() {
        // ... загрузка
        didFinishLoading.send(())
    }
}

class ViewController {
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        customView.didFinishLoading
            .sink { [weak self] _ in
                self?.hideLoadingIndicator()
            }
            .store(in: &cancellables)
    }
}
```

#### 4. PassthroughSubject + combineLatest (очень мощно)

```swift
let refreshTrigger = PassthroughSubject<Void, Never>()
let filterChanged = PassthroughSubject<String, Never>()

refreshTrigger
    .combineLatest(filterChanged)
    .flatMap { _, filter in
        fetchItems(with: filter)
    }
    .assign(to: &$items)
```

#### 5. PassthroughSubject для глобальных событий (Event Bus)

```swift
enum AppEvent {
    case userDidLogin(User)
    case themeChanged(Theme)
    case logout
}

class EventBus {
    static let shared = EventBus()
    
    let events = PassthroughSubject<AppEvent, Never>()
    
    private init() {}
}

EventBus.shared.events
    .sink { event in
        switch event {
        case .userDidLogin(let user): print("Вход:", user.name)
        case .themeChanged(let theme): applyTheme(theme)
        case .logout: handleLogout()
        }
    }
    .store(in: &globalCancellables)

// где угодно в приложении
EventBus.shared.events.send(.userDidLogin(currentUser))
```

### Сравнение PassthroughSubject с другими Publisher

| Publisher                     | Хранит значение? | Начальное значение | Когда подписчик получает данные                  | Когда использовать в 2026 |
|-------------------------------|------------------|--------------------|--------------------------------------------------|----------------------------|
| `PassthroughSubject`          | Нет              | Нет                | Только после подписки                            | события, команды, broadcast |
| `CurrentValueSubject`         | Да               | Да (обязательно)   | Сразу получает текущее + новые                   | состояние с текущим значением |
| `@Published`                  | Да               | Да (инициализация) | Сразу получает текущее + новые                   | ViewModel → View в MVVM |
| `Just(value)`                 | Да               | Да                 | Один раз и завершается                           | фиксированное значение |
| `Empty(completeImmediately:)` | Нет              | Нет                | Ничего не получает, сразу завершается            | пустой успешный результат |

### Лучшие практики PassthroughSubject в Combine 2026

- **Используйте** `PassthroughSubject<Void, Never>` для простых событий (нажатие, dismiss, reload)  
- **Добавляйте тип ошибки**, если может быть сбой (например `PassthroughSubject<User, AuthError>`)  
- **Храните** в `private let` или `private(set) var` — обычно не меняют после создания  
- **Не храните состояние** — если нужно текущее значение → `CurrentValueSubject` или `@Published`  
- **Для глобальных событий** — создавайте singleton (EventBus) или инжектируйте через DI  
- **В SwiftUI** — PassthroughSubject редко нужен — используйте `@Published` или `@Observable`  
- **Документируйте** — всегда пиши комментарий:

```swift
/// Событие выбора элемента в списке
let itemSelected = PassthroughSubject<String, Never>()
```

**Короткий итог 2026**:
> `PassthroughSubject` — это **Publisher без памяти**: не хранит значение, передаёт события только текущим подписчикам.  
> В 2026 году:  
> - идеален для **событий**, **команд**, **уведомлений**, **broadcast**  
> - не имеет начального значения — новые подписчики ничего не получают сразу  
> - стандартный выбор для замены делегатов, NotificationCenter, кастомных событий  
> - используй `Void` для простых триггеров, конкретный тип для передачи данных  

Удачи с чистыми и предсказуемыми событиями в твоём проекте! 📡