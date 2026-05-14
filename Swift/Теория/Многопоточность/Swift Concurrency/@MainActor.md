**`@MainActor`** — это **встроенный глобальный актёр** (global actor), который гарантирует:

- весь код, помеченный этим атрибутом, будет выполняться **строго на главном потоке** ([[main|main thread]] / UI [[thread]])  
- компилятор автоматически вставляет переключение контекста (hop to main actor)  
- любые попытки доступа к изолированным свойствам/методам без правильной изоляции приводят к **ошибке компиляции** в строгом режиме ([[Swift]] 6+)

**Коротко и по-человечески**:
> `@MainActor` = «всё, что здесь написано, должно выполняться на главном потоке — и точка».

Это **самый важный** и **самый часто используемый** глобальный актёр в [[iOS]]/macOS-разработке 2025–2026 годов.

### 2. Почему @MainActor так важен

| Проблема без @MainActor                    | Последствия                             | Как @MainActor решает проблему     |
| ------------------------------------------ | --------------------------------------- | ---------------------------------- |
| Обновление UI из фонового потока           | [[Main Thread Violation]] → краш / баги | Гарантированное выполнение на main |
| Забыть `DispatchQueue.main.async`          | Случайные краши, визуальные артефакты   | Компилятор ловит ошибку заранее    |
| Множество `await DispatchQueue.main.async` | Код становится нечитаемым               | Один `@MainActor` — и всё чисто    |
| Сложные цепочки Task + UI-обновление       | Легко забыть переключение на main       | Изоляция на уровне типа/метода     |

**Факт 2026**:
> В Swift 6+ с включённой полной проверкой строгой конкурентности (`strict concurrency checking = complete`) почти **все** UI-классы и ViewModel **обязаны** быть помечены `@MainActor` — иначе компилятор выдаст ошибку.

### 3. Все способы применения @MainActor (2026 актуально)

| Способ применения                    | Что происходит                                | Самый частый use-case в 2026                      |
| ------------------------------------ | --------------------------------------------- | ------------------------------------------------- |
| `@MainActor` на всём классе / struct | Все методы и свойства изолированы на main     | ViewModel, ObservableObject, [[UIViewController]] |
| `@MainActor` на отдельном методе     | Только этот метод выполняется на main         | Отдельные UI-обновления из actor                  |
| `@MainActor` на свойстве             | Доступ к свойству требует [[main]]-контекста  | Редко (лучше на классе)                           |
| `@MainActor func` в обычном классе   | Метод требует `await` при вызове из фона      | Сервисы с UI-действиями                           |
| `await MainActor.run { ... }`        | Временное переключение на main внутри функции | Когда не хочется аннотировать весь метод          |
| `Task { @MainActor in ... }`         | Создаёт задачу, которая сразу на main         | Запуск UI-логики из фона                          |

### 4. Самые популярные шаблоны 2026 года

#### Шаблон 1 — ViewModel целиком на @MainActor (самый частый)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    @Published var isLoading = false
    
    func load() async {
        isLoading = true
        let fetched = try? await fetchProfile()
        profile = fetched
        isLoading = false
    }
}
```

#### Шаблон 2 — Смешивание @MainActor и [[actor]]

```swift
actor DataLoader {
    func loadProfile() async -> Profile {
        // фоновая работа
        return try await api.fetchProfile()
    }
}

@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    
    let loader = DataLoader()
    
    func refresh() {
        Task {
            let data = await loader.loadProfile()
            profile = data  // безопасно — мы уже на MainActor
        }
    }
}
```

#### Шаблон 3 — Временное переключение с [[await]] MainActor.run

```swift
actor Analytics {
    func trackEvent(_ name: String) async {
        let payload = await preparePayload()
        
        await MainActor.run {
            // только здесь можно безопасно вызвать UI-аналитику
            AnalyticsManager.shared.logEvent(name, payload: payload)
        }
    }
}
```

#### Шаблон 4 — @MainActor на замыкании / функции

```swift
func showAlert(message: String) {
    Task { @MainActor in
        let alert = UIAlertController(title: "Сообщение", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        UIApplication.shared.keyWindow?.rootViewController?.present(alert, animated: true)
    }
}
```

### 5. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `@MainActor` на ViewModel            | Ошибка компиляции в Swift 6+             | Всегда помечать ObservableObject |
| Вызов метода @MainActor без await из фона   | Ошибка компиляции                        | Всегда `await` при вызове извне |
| `Task { ... }` внутри @MainActor без изоляции | Потеря контекста → ошибка                | Использовать `@_inheritActorContext` (редко) или `Task { @MainActor in ... }` |
| Синхронный вызов сети / I/O на @MainActor   | UI freeze (ANR)                          | Всё тяжёлое — в обычный `actor` или `Task.detached` |
| `@MainActor` на очень большом классе        | Всё становится последовательным → лаги   | Только UI-логика на @MainActor, данные — в actor |

### 6. @MainActor vs другие механизмы изоляции (2026 сравнение)

| Механизм                 | Поток выполнения  | Глобальность | Отмена задач   | Сложность | Рекомендация 2026      |
| ------------------------ | ----------------- | ------------ | -------------- | --------- | ---------------------- |
| `@MainActor`             | Главный поток     | Да           | Через [[Task]] | Низкая    | Для всего UI           |
| `@globalActor` (свой)    | Любой (по выбору) | Да           | Через Task     | Средняя   | Для БД, логов          |
| обычный `actor`          | Произвольный      | Нет          | Через Task     | Низкая    | Данные / бизнес-логика |
| `@isolated(any)`         | От параметра      | Нет          | Через Task     | Средняя   | Универсальные функции  |
| DispatchQueue.main.async | Главный поток     | Да           | Нет            | Средняя   | Legacy-код             |

**Вывод 2026**:
- `@MainActor` — **единственный правильный** способ работы с UI в Swift 6+  
- Всё остальное состояние → обычные `actor`  
- Глобальные актёры (кроме `@MainActor`) → только для очень специфических случаев

### 7. Лучшие практики 2026 года

- **Все ViewModel / ObservableObject** — помечать `@MainActor`  
- **Все UIViewController** — помечать `@MainActor` (в [[SwiftUI]]/[[UIKit]])  
- **Тяжёлая работа** — выносить в обычный `actor` или `Task.detached`  
- **Вызовы из фона** — всегда через `await` или `Task { @MainActor in ... }`  
- **Swift 6 strict concurrency** — включать полную проверку — ловит почти все ошибки  
- **Тестирование** — использовать `await MainActor.run` в тестах  
- **Мониторинг** — включать **Main Thread Checker** в схеме Xcode

**Короткий девиз 2026**:
> «@MainActor — это когда ты говоришь: «всё, что здесь, должно быть на главном потоке — и точка».  
> В 2026 году это уже не рекомендация, а **обязательное правило** для всего UI-кода в Swift.»
