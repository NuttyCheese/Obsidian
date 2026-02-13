### 1. Что такое DispatchQueue.main (реальность 2026 года)

`DispatchQueue.main` — это **единственная системная очередь**, которая **всегда привязана к главному потоку** (main thread / UI thread) приложения.

Это **самая важная очередь** в [[iOS]]/macOS-приложении, потому что:

- **только** в ней можно безопасно обновлять пользовательский интерфейс ([[UIKit]], [[SwiftUI]], [[AppKit]])  
- все операции с UI (изменение текста, цвета, перезагрузка таблиц, анимации, present, push и т.д.) **должны** выполняться именно здесь  
- очередь **сериальная** ([[serial]]) — задачи выполняются строго по порядку добавления  
- она **никогда не блокируется** самой системой (кроме случаев [[deadlock]] от неправильного [[sync]])

**Ключевые факты 2026 года**:

- `DispatchQueue.main` = `DispatchQueue.main` (статическая переменная)  
- Это **не глобальная очередь** — это **очередь главного потока**  
- В [[Swift Concurrency]] она **полностью совместима** с `@MainActor`  
- В Swift 6 strict concurrency checking — попытка обновить UI без `@MainActor` или `main.async` часто вызывает ошибку компиляции

### 2. Когда и зачем использовать DispatchQueue.main

| Сценарий                                             | Почему нужен DispatchQueue.main    | Альтернатива в 2026 году (рекомендуемая)            |
| ---------------------------------------------------- | ---------------------------------- | --------------------------------------------------- |
| Обновление [[UILabel]].text / [[UIButton]].setTitle  | UIKit не потокобезопасен           | [[@MainActor]] / [[await]] MainActor.run            |
| tableView.reloadData() / collectionView.reloadData() | То же самое                        | @MainActor                                          |
| [[UIView]].animate / CAAnimation                     | Анимации привязаны к main run loop | @MainActor                                          |
| present(_:animated:completion:)                      | Модальные окна только в main       | @MainActor                                          |
| [[UINavigationController]].pushViewController        | Навигация только в main            | @MainActor                                          |
| Изменение @Published / @State в SwiftUI              | [[SwiftUI]] требует main thread    | @MainActor                                          |
| Обработка уведомлений [[NotificationCenter]]         | Часто приходят из фона             | [[DispatchQueue]].[[main]].[[async]] или @MainActor |

### 3. Самые частые и правильные шаблоны использования в 2026

#### Шаблон 1 — Фон → UI (самый частый паттерн)

```swift
func loadUserProfile() {
    // Фоновая загрузка (сеть, диск, Core Data)
    DispatchQueue.global(qos: .userInitiated).async {
        let profile = fetchProfileFromNetwork()
        
        DispatchQueue.main.async {
            // Только UI-обновление
            self.avatarImageView.image = profile.avatar
            self.nameLabel.text = profile.name
            self.tableView.reloadData()
        }
    }
}
```

#### Шаблон 2 — Современный (Swift Concurrency + @MainActor)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    
    func load() async {
        let fetched = try await fetchProfile()
        profile = fetched  // безопасно — @MainActor
    }
}

// или без @MainActor на классе
func load() async {
    let fetched = try await fetchProfile()
    
    await MainActor.run {
        self.profile = fetched
        self.tableView.reloadData()
    }
}
```

#### Шаблон 3 — Обработка уведомлений / [[callback]] из C [[API]]

```swift
NotificationCenter.default.addObserver(forName: .userDidLogin, object: nil, queue: .main) { notification in
    // безопасно — уже в main
    self.updateUIAfterLogin()
}
```

### 4. Самые опасные ошибки с DispatchQueue.main в 2026

| Ошибка                                          | Последствия                                               | Как избежать                                                  |
| ----------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------- |
| `DispatchQueue.main.sync` из главного потока    | [[Deadlock]] / зависание UI                               | Никогда не использовать `sync` на main                        |
| Обновление UI из `Task.detached` без @MainActor | [[Main Thread Violation]] (в Swift 6 — ошибка компиляции) | Всегда `@MainActor` или `MainActor.run`                       |
| `DispatchQueue.main.async` внутри @MainActor    | Ненужный overhead (задача откладывается)                  | Просто делай UI-обновления напрямую                           |
| Синхронный вызов сети / I/O на main             | UI freeze (ANR)                                           | Всегда в `.global` или `Task`                                 |
| Забыть перейти на main после фона               | [[Main Thread Violation]]                                 | Всегда проверять: `DispatchQueue.main.async` или `@MainActor` |

### 5. DispatchQueue.main vs Swift Concurrency (честное сравнение 2026)

| Характеристика              | DispatchQueue.main          | Swift Concurrency (@MainActor / MainActor.run) | Что выбрать в 2026 году    |
| --------------------------- | --------------------------- | ---------------------------------------------- | -------------------------- |
| Потокобезопасность UI       | Ручная ([[async]]/[[sync]]) | Встроенная                                     | @MainActor                 |
| Читаемость                  | Старый стиль                | Современный, декларативный                     | @MainActor                 |
| Отмена задач                | Нет                         | Нативная (Task.cancel)                         | Swift Concurrency          |
| Strict Concurrency Checking | Частично (может молчать)    | Полностью (ошибки компиляции)                  | Swift Concurrency          |
| Совместимость с legacy      | Отличная                    | Требует адаптации                              | @MainActor для нового кода |

**Вывод 2026**:
- **Новый код** → **@MainActor** / `await MainActor.run` / `Task { @MainActor in ... }`  
- **Поддержка старого кода** → `DispatchQueue.main.async`  
- `DispatchQueue.main.sync` — почти всегда ошибка (deadlock)

### 6. Лучшие практики 2026 года

- **Переходите на Swift 6** — strict concurrency checking ловит почти все Main Thread Violation  
- **@MainActor** — всё, что касается UI, SwiftUI, ViewModel, UIViewController  
- **actor** — для бизнес-логики и данных  
- **Sendable** — всё, что передаётся между акторами / потоками  
- **Никогда** не вызывайте UI-код из `Task.detached`, `DispatchQueue.global()`, сетевых completion  
- **Все сетевые/тяжёлые операции** — в `Task` или `actor`, UI-обновления — через `@MainActor`  
- **Для legacy-кода** — оборачивайте в `@MainActor` или `DispatchQueue.main.async`  
- **В тестах** — используйте `await MainActor.run` или `@MainActor` в тестовых классах  
- **Избегайте** `DispatchQueue.main.sync` — это почти всегда deadlock  
- **Мониторинг** — включайте **Main Thread Checker** в схеме Xcode (Diagnostics)

**Короткий девиз 2026**:
> «DispatchQueue.main — это когда нужно обновить UI из фона.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — @MainActor + await MainActor.run + Swift 6 strict concurrency.  
> Если ты всё ещё пишешь .main.async в новом коде — спроси себя: «А точно ли это нужно?»»
