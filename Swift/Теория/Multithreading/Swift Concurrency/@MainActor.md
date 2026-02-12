## 1. Что такое `@MainActor`

`@MainActor` — это **глобальный actor**, встроенный в [[Swift]].

👉 Он гарантирует, что код, помеченный этим атрибутом, будет выполняться **на главном потоке (main thread)**.  
А главный поток — это именно то место, где должен выполняться **UI-код ([[UIKit]], SwiftUI)**.

По сути, [[@MainActor]] = "Убедиcь, что это выполняется в UI-потоке".

---

## 2. Как это работает

- Если вы пометили метод, свойство или класс `@MainActor`, то [[Swift]] Concurrency обеспечит **изолированное выполнение** на главном потоке.
    
- Вызов такого метода из фонового потока будет требовать [[await]], чтобы переключиться на `MainActor`.
    
- Это избавляет от ручного `DispatchQueue.main.async { ... }`, которым пользовались раньше.
    

---

## 3. Простые примеры

### Пример 1. Метод на главном потоке

```swift
@MainActor
func updateUI() {
    print("Выполняется на главном потоке:", Thread.isMainThread) // true
}
```

---

### Пример 2. Класс с аннотацией

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var text: String = "Привет"

    func changeText() {
        text = "Обновлено ✅"
    }
}
```

👉 Здесь **все свойства и методы** выполняются на `MainActor`.

---

### Пример 3. Вызов из фонового потока

```swift
@MainActor
func showMessage() {
    print("Сообщение на UI")
}

Task {
    print("Фоновая задача")
    await showMessage() // переключение на главный поток
}
```

---

### Пример 4. SwiftUI + @MainActor

```swift
@MainActor
class AppState: ObservableObject {
    @Published var status: String = "Загрузка..."
    
    func updateStatus() {
        status = "Готово ✅"
    }
}

struct ContentView: View {
    @StateObject private var state = AppState()

    var body: some View {
        VStack {
            Text(state.status)
            Button("Обновить") {
                Task {
                    await state.updateStatus()
                }
            }
        }
    }
}
```

👉 `AppState` полностью живёт на `MainActor`, так что обновление UI безопасно.

---

### Пример 5. Смешивание с фоновой работой

```swift
actor DataLoader {
    func load() async -> String {
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return "Данные загружены"
    }
}

@MainActor
class ViewModel: ObservableObject {
    @Published var text = "Загрузка..."

    let loader = DataLoader()

    func fetchData() {
        Task {
            let result = await loader.load()
            text = result // выполняется на MainActor
        }
    }
}
```

👉 загрузка данных идёт в actor (фоново), а обновление текста — на главном потоке.

---

### Пример 6. Сравнение с GCD

**Раньше (до Swift Concurrency):**

```swift
DispatchQueue.global().async {
    let data = "Загружено"
    DispatchQueue.main.async {
        self.label.text = data
    }
}
```

**Теперь (с `@MainActor`):**

```swift
Task {
    let data = "Загружено"
    await updateLabel(with: data)
}

@MainActor
func updateLabel(with text: String) {
    label.text = text
}
```

👉 Код становится чище и безопаснее.

---

## 4. Когда использовать `@MainActor`

- Для **UI-логики** (обновление интерфейса).
    
- Для `ObservableObject` в SwiftUI.
    
- Для сервисов, где доступ к данным должен быть строго из одного места (например, глобальный state приложения).
    
- Чтобы избежать ошибок "модификация UI не с main thread".
    

---

## 5. Сравнение с другими атрибутами

|Атрибут|Что делает|
|---|---|
|`@MainActor`|Гарантирует выполнение на главном потоке (UI)|
|`@globalActor`|Позволяет создать свой глобальный actor (например, для базы данных)|
|`@isolated(any)`|Позволяет функции унаследовать изоляцию от переданного actor-параметра|
|`@_inheritActorContext`|Внутренний атрибут (нестабильный), сохраняет actor-контекст в Task|

---

## 6. Выводы

- `@MainActor` — это встроенный глобальный actor для UI.
    
- Помеченные им методы/классы всегда выполняются на **главном потоке**.
    
- Это замена старого `DispatchQueue.main.async`.
    
- Работает как гарант безопасного доступа к UI.
    

---
