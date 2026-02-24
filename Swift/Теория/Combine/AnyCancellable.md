**`AnyCancellable`** — это тип из фреймворка **[[Combine]]**, который представляет **одноразовый токен подписки** на `Publisher`.

Его главная задача — **автоматически отменять подписку**, когда токен выходит из области видимости ([[deinit]]) или когда явно вызывается метод `.cancel()`.

Это **единственный** официальный и рекомендуемый способ управлять жизненным циклом подписок в Combine.

### Почему именно AnyCancellable, а не что-то другое?

| Альтернатива                                   | Проблемы / недостатки в 2025–2026                                                            | Почему AnyCancellable лучше                            |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Хранить `Cancellable` напрямую                 | Разные [[Publisher]] возвращают разные типы Cancellable (Publishers.Merge.Cancellable, etc.) | AnyCancellabe — type-erased, единый тип                |
| Использовать DisposeBag (как в [[RxSwift]])    | Нужно отдельная библиотека или писать свою реализацию                                        | Combine имеет встроенное решение                       |
| Забывать отменять подписку                     | → [[retain cycle]], утечки памяти, лишние сетевые запросы                                    | AnyCancellable + .store(in:) — почти невозможно забыть |
| Использовать [[weak]] [[self]] + manual cancel | Много boilerplate, легко ошибиться                                                           | .store(in: &cancellables) — одна строка                |

### Основные способы хранения AnyCancellable (2026 стандарт)

#### 1. Самый популярный и рекомендуемый — Set<AnyCancellable>

```swift
class ViewModel {
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // Подписка на @Published
        $someProperty
            .sink { [weak self] value in
                self?.updateUI(with: value)
            }
            .store(in: &cancellables)  // ← магия здесь
        
        // Сетевая подписка
        URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: [User].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink(receiveCompletion: { completion in
                // обработка ошибок
            }, receiveValue: { [weak self] users in
                self?.users = users
            })
            .store(in: &cancellables)
    }
}
```

**Почему Set<AnyCancellable> — золотой стандарт:**
- автоматически отменяет все подписки при deinit ViewModel
- одна строка `.store(in: &cancellables)`
- работает с любым типом Publisher
- thread-safe для большинства случаев

#### 2. Хранение в массиве (редко, но встречается)

```swift
private var cancellables: [AnyCancellable] = []
```

Минус: нельзя использовать `Set`, если нужно хранить несколько одинаковых подписок (редко).

#### 3. Хранение в свойстве (для одной подписки)

```swift
private var subscription: AnyCancellable?

func subscribe() {
    subscription = publisher
        .sink { value in ... }
}
```

Минус: нужно вручную отменять при необходимости.

### Полный современный пример ViewModel (2026 стиль)

```swift
import Combine
import Foundation

@MainActor
class UserListViewModel: ObservableObject {
    
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    
    private let service: UserService
    
    init(service: UserService = .live) {
        self.service = service
        
        // Автоматическая загрузка при создании
        loadUsers()
    }
    
    func loadUsers() {
        isLoading = true
        errorMessage = nil
        
        service.fetchUsers()
            .receive(on: DispatchQueue.main)
            .sink { [weak self] completion in
                self?.isLoading = false
                if case .failure(let error) = completion {
                    self?.errorMessage = error.localizedDescription
                }
            } receiveValue: { [weak self] users in
                self?.users = users
            }
            .store(in: &cancellables)
    }
    
    func refresh() {
        // Можно добавить debounce, если нужно
        loadUsers()
    }
}
```

### Лучшие практики AnyCancellable в Swift 2026

- **Всегда** храните подписки в `Set<AnyCancellable>` или `[AnyCancellable]`  
- **Используйте** `private var cancellables = Set<AnyCancellable>()` в ViewModel / Controller  
- **Никогда** не храните `AnyCancellable` в глобальных переменных — это приведёт к утечкам  
- **В SwiftUI** — подписки обычно живут в `@StateObject` / `@ObservableObject` → автоматически отменяются при исчезновении View  
- **Для долгоживущих подписок** (например, глобальный EventBus) — храните в singleton с явной отменой  
- **Для Combine + async/await** — часто подписки заменяют на `Task` → AnyCancellable не нужен  
- **Документируйте** — пишите комментарий «Хранилище всех Combine-подписок ViewModel — отменяются автоматически при deinit»

**Короткий итог 2026**:
> `AnyCancellable` — это **токен подписки** в Combine, который **автоматически отменяет** подписку при своём deinit.  
> В 2026 году:  
> - стандартный паттерн — `.store(in: &cancellables)`  
> - хранилище — `private var cancellables = Set<AnyCancellable>()`  
> - предотвращает утечки памяти и ненужные сетевые запросы  
> - это **единственный рекомендуемый** способ управлять подписками в Combine  
