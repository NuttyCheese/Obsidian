**Global GCD** (или **global dispatch queues**) — это набор предопределённых очередей в **Grand Central Dispatch** ([[GCD]]), которые Apple предоставляет «из коробки» и которые используются почти в каждом iOS/macOS-приложении.

Вот актуальное и полное состояние global queues на 2026 год ([[Swift]] 6 / [[iOS]] 18+ / macOS 15+).

### 1. Список всех глобальных очередей ([[QoS]])

| Очередь                          | QoS-параметр              | Приоритет (относительный) | Когда использовать (2026 рекомендации)                     | Современная альтернатива в Swift Concurrency |
|----------------------------------|----------------------------|----------------------------|-------------------------------------------------------------|-----------------------------------------------|
| `.global(qos: .userInteractive)` | User Interactive           | ★★★★★ (самый высокий)      | Анимации, скролл, жесты, реакция на касания                | `Task(priority: .userInteractive)`            |
| `.global(qos: .userInitiated)`   | User Initiated             | ★★★★☆                      | Запуск после нажатия кнопки, загрузка данных по действию   | `Task(priority: .userInitiated)`              |
| `.global()` / `.global(qos: .default)` | Default              | ★★★☆☆                      | Большинство фоновых задач без явного приоритета             | `Task {}` (по умолчанию)                      |
| `.global(qos: .utility)`         | Utility                    | ★★☆☆☆                      | Долгие операции: обработка фото, индексация, загрузка файлов | `Task(priority: .utility)`                    |
| `.global(qos: .background)`      | Background                 | ★☆☆☆☆ (самый низкий)       | Фоновые задачи: синхронизация, аналитика, чистка кэша      | `Task(priority: .background)`                 |

### 2. Как правильно использовать global queues в 2026 году

#### Рекомендуемый стиль (современный + безопасный)

```swift
// Самый частый и рекомендуемый паттерн 2026
Task(priority: .userInitiated) {
    let data = await heavyNetworkCall()
    
    await MainActor.run {
        self.tableView.reloadData(with: data)
    }
}

// Или более явный вариант с QoS
DispatchQueue.global(qos: .userInitiated).async {
    let data = heavyComputation()
    
    DispatchQueue.main.async {
        self.updateUI(with: data)
    }
}
```

#### Правила хорошего тона 2026

- **Всегда указывай QoS явно** — `.default` почти никогда не нужен  
- **Не используй .sync на global-очередях** — это бессмысленно и опасно  
- **Не делай тяжёлую работу на .main** — только UI-обновления  
- **Переходи на Task { } / Task(priority:) вместо DispatchQueue.global()**  
  → Task автоматически наследует приоритет и контекст изоляции  
  → Лучше читается и меньше boilerplate

### 3. Сравнение DispatchQueue.global vs Task (2026)

| Критерий                                 | DispatchQueue.global(qos:)             | Task(priority:)                   | Победитель 2026 |
| ---------------------------------------- | -------------------------------------- | --------------------------------- | --------------- |
| Читаемость                               | ★★★☆☆                                  | ★★★★★                             | Task            |
| Поддержка [[async]]/[[await]]            | Только через completion / continuation | Нативная                          | Task            |
| Автоматическое переключение на MainActor | Нет                                    | Да (если родитель [[@MainActor]]) | Task            |
| Наследование приоритета                  | Нет                                    | Да                                | Task            |
| Отмена задачи                            | Нет встроенной                         | `task.cancel()`                   | Task            |
| Strict Concurrency (Swift 6)             | Требует ручной проверки                | Полная поддержка                  | Task            |
| Производительность                       | Чуть выше в горячих петлях             | Почти идентична                   | Ничья           |

**Вывод**:  
В 2026 году **DispatchQueue.global()** используют только в двух случаях:

1. Нужна **очень тонкая настройка QoS** в горячих петлях (редко)  
2. Поддержка старого кода / библиотек, которые ещё не мигрированы на async/await

Во всех остальных ситуациях → **Task { } / Task(priority:)**

### 4. Лучшие практики работы с global GCD в 2026

- **Переходи на Task** — это уже стандарт  
- **QoS указывай всегда** — `.userInitiated` для большинства пользовательских действий  
- **Не используй .sync на global-очередях** — это бессмысленно и опасно  
- **Для UI** — только `@MainActor` / `MainActor.run` / `await MainActor.run`  
- **Для фоновых вычислений** — `Task.detached(priority: .utility)`  
- **Для CPU-интенсивных задач** — `Task.detached(priority: .background)` + `actor`  
- **Swift 6 strict concurrency** — избегай захвата [[self]] в global-замыканиях без `[weak self]`

**Короткий девиз 2026**:
> «Global GCD в 2026 году — это как дискета в 2025: работает, но почти никто не использует.  
> Основной инструмент — Task { } / Task(priority:).  
> DispatchQueue.global() оставь только для очень специфических случаев и поддержки legacy-кода.»
