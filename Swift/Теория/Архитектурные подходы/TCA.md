**TCA** (The Composable Architecture) — это современная архитектура для приложений на Swift, разработанная компанией Point-Free.

К 2026 году это **самая популярная и рекомендуемая** архитектура для новых [[iOS]]-приложений на [[SwiftUI]] + [[Swift Concurrency]].

### Кратко: что такое TCA

TCA — это **унифицированная, предсказуемая и тестируемая** архитектура, основанная на принципах:

- Unidirectional data flow (однонаправленный поток данных)  
- State + Action + Reducer  
- Композиция (очень сильная сторона)  
- Полная тестируемость (включая эффекты)  
- Встроенная поддержка Swift Concurrency ([[async]]/[[await]], [[actor]], [[Task]] и т.д.)

### Основные блоки TCA (2026 версия)

| Компонент       | Что это простыми словами                              | Тип в Swift 6+                  | Самая важная черта 2026 |
|-----------------|-------------------------------------------------------|----------------------------------|--------------------------|
| **State**       | Всё, что отображается на экране + внутреннее состояние | struct (обычно) / @Observable   | Полностью иммутабельный |
| **Action**      | Всё, что может произойти (нажатие, появление экрана, ответ сервера) | enum с associated values        | Закрытый набор событий |
| **Reducer**     | Функция: (inout State, Action) → Effect               | (inout State, Action) → Effect  | Чистая + предсказуемая |
| **Effect**      | Асинхронная побочная операция (запрос, таймер, уведомление) | Effect<Action>                  | Управляет всеми side-effects |
| **Store**       | Центральный объект: хранит State и принимает Action   | Store<State, Action>            | Один на фичу / экран     |
| **View**        | SwiftUI View, которая читает State и отправляет Action | View + WithViewStore / @Bindable | Минимальная логика       |

### Самый простой и актуальный пример TCA в 2026 году

```swift
import ComposableArchitecture

// 1. State
@ObservableState
struct CounterState {
    var count = 0
    var isLoading = false
}

// 2. Action
enum CounterAction {
    case increment
    case decrement
    case fetchFact
    case factResponse(Result<String, Error>)
}

// 3. Reducer
@Reducer
struct CounterFeature {
    @ObservableState
    struct State { ... } // как выше
    
    enum Action { ... } // как выше
    
    @Dependency(\.numberFactClient) var numberFactClient
    
    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .increment:
                state.count += 1
                return .none
                
            case .decrement:
                state.count -= 1
                return .none
                
            case .fetchFact:
                state.isLoading = true
                return .run { send in
                    let fact = try await numberFactClient.fetch(state.count)
                    await send(.factResponse(.success(fact)))
                } catch: { error in
                    await send(.factResponse(.failure(error)))
                }
                
            case .factResponse(.success(let fact)):
                state.isLoading = false
                // можно показать alert или sheet с фактом
                return .none
                
            case .factResponse(.failure):
                state.isLoading = false
                return .none
            }
        }
    }
}

// 4. SwiftUI View
struct CounterView: View {
    @Bindable var store: StoreOf<CounterFeature>
    
    var body: some View {
        VStack {
            Text("Count: \(store.count)")
            
            HStack {
                Button("−") { store.send(.decrement) }
                Button("+") { store.send(.increment) }
            }
            
            if store.isLoading {
                ProgressView()
            }
            
            Button("Get fact") {
                store.send(.fetchFact)
            }
        }
        .padding()
    }
}
```

### Почему TCA доминирует в 2026 году

| Преимущество                      | Почему это важно в 2026 году                               |
| --------------------------------- | ---------------------------------------------------------- |
| Полная предсказуемость            | Один источник истины (State + Action)                      |
| Композиция на уровне reducer’ов   | Легко собирать большие приложения из маленьких фич         |
| Встроенная поддержка async/await  | Эффекты как `Effect.run` / `.run`                          |
| Отличная тестируемость            | Тесты reducer’ов — чистые функции                          |
| Отмена операций                   | `store.send(.cancel)` + `withTaskCancellation`             |
| Swift 6 strict concurrency        | Полная поддержка [[Sendable]] / [[actor]] / [[@MainActor]] |
| SwiftUI + @Bindable + @Observable | Идеальная интеграция с SwiftUI 2024–2026                   |
| A/B-тестирование и feature flags  | Легко включать/выключать фичи через reducer                |

### Самые популярные дополнения к TCA в 2026

- **swift-dependencies** — встроенный DI (Dependency Values)  
- **swift-case-paths** — мощные паттерны для работы с enum  
- **swift-navigation** — навигация (Sheet, NavigationStack, Alert)  
- **swiftui-navigation** — typed navigation (Point-Free)  
- **composable-architecture-testing** — тесты с `TestStore`  
- **sourcery** / **swift-gen** — генерация boilerplate для State/Action

### Когда TCA действительно стоит использовать (рекомендация 2026)

| Размер проекта / сложность                      | Рекомендация TCA      | Альтернатива                                                     |
| ----------------------------------------------- | --------------------- | ---------------------------------------------------------------- |
| Маленькое приложение (1–3 экрана)               | Можно обойтись        | [[MVVM (Model-View-ViewModel) Architecture\|MVVM]] + @Observable |
| Среднее приложение (5–15 экранов)               | Сильно рекомендуется  | MVVM-C / [[Clean Swift (VIP) Architecture\|Clean Swift]]         |
| Большое приложение (20+ экранов, команда 5+)    | **Обязательно**       | —                                                                |
| Приложение с сильным состоянием (формы, wizard) | ★★★★★                 | —                                                                |
| Проект с долгой жизнью (5–10+ лет)              | ★★★★★                 | —                                                                |
| A/B-тестирование, feature flags                 | ★★★★★                 | —                                                                |
| Очень простые экраны (статичные списки)         | Можно не использовать | SwiftUI + @State                                                 |

**Короткий вердикт 2026**:
> «TCA в 2026 году — это когда ты хочешь максимальную предсказуемость, тестируемость и масштабируемость.  
> Для большинства новых iOS-приложений средней и большой сложности — это **лучший выбор**.  
> Для очень маленьких приложений или прототипов — можно обойтись чистым SwiftUI + MVVM.»
