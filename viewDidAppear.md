**viewDidAppear(_:)** — это один из самых важных методов жизненного цикла **UIViewController** в UIKit.

Он вызывается **каждый раз**, когда контроллер представления **становится видимым** на экране (появляется в иерархии окон).

### Когда вызывается viewDidAppear (2026 актуально)

| Событие / Действие                            | viewDidAppear вызывается? | Примечание |
|-----------------------------------------------|----------------------------|------------|
| Первый показ экрана (push / present)          | **Да**                     | Самый первый вызов после viewDidLoad |
| Возврат с предыдущего экрана (pop / dismiss)  | **Да**                     | Каждый раз при возврате |
| Переход с tab bar (переключение вкладки)      | **Да**                     | При каждом выборе вкладки |
| Экран был скрыт (например, модальное окно сверху) и снова показан | **Да**                     | Повторный показ |
| Экран повёрнут (orientation change)           | **Нет** (если не был скрыт) | Обычно viewWillTransition(to:with:) |
| Приложение вернулось из фона (foreground)     | **Да** (если контроллер был видимым) | UIApplication.willEnterForegroundNotification |
| Контроллер добавлен как child                 | **Да**                     | addChild → didMove(toParent:) |

### Порядок вызовов жизненного цикла (самая частая последовательность)

```text
1. init / init(coder:)
2. loadView() / viewDidLoad()
3. viewWillAppear(_:)
4. viewDidAppear(_:)      ← здесь ты обычно
5. viewWillDisappear(_:)
6. viewDidDisappear(_:)
```

### Что обычно делают в viewDidAppear в 2026 году

| Действие                                      | Почему именно здесь (а не в viewDidLoad) | Пример кода (современный стиль) |
|-----------------------------------------------|-------------------------------------------|---------------------------------|
| Запуск сетевых запросов / загрузка данных     | Экран уже виден → пользователь видит прогресс | `Task { await loadData() }` |
| Обновление UI при возврате с другого экрана   | Данные могли измениться (pop / dismiss)   | `updateUIWithLatestData()` |
| Регистрация на уведомления / наблюдатели      | Нужно обновлять при каждом появлении      | `NotificationCenter.default.addObserver(...)` |
| Запуск анимаций / скролл / фокус              | Экран уже отрисован → анимации плавные    | `tableView.scrollToRow(at: ..., animated: true)` |
| Отслеживание аналитики (screen view)          | Точное время показа экрана                | `Analytics.trackScreen("Profile")` |
| Настройка фокуса / first responder            | Экран видим → можно показать клавиатуру   | `textField.becomeFirstResponder()` |
| Проверка авторизации / deep link              | При каждом появлении проверять состояние  | `if !isAuthorized { showLogin() }` |

### Самый современный паттерн 2026 (async/await + @MainActor)

```swift
@MainActor
class ProfileViewController: UIViewController {
    
    private let viewModel: ProfileViewModel
    
    init(viewModel: ProfileViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        // Самый безопасный и современный способ
        Task {
            await refreshData()
        }
        
        // Или без Task, если не нужна асинхронность
        updateAnalytics()
    }
    
    private func refreshData() async {
        do {
            try await viewModel.loadProfile()
            updateUI()
        } catch {
            showError(error)
        }
    }
    
    private func updateAnalytics() {
        Analytics.trackScreen("Profile", properties: ["userId": viewModel.userId])
    }
    
    private func updateUI() {
        // обновление UILabel, UITableView и т.д.
    }
}
```

### Лучшие практики viewDidAppear в Swift 2026

- **Вызывай super.viewDidAppear(animated)** — **обязательно** в начале  
- **Не делай тяжёлую работу** — максимум 1–2 мс (иначе дропы кадров)  
- **Async/await** — оборачивай сетевые/долгие операции в `Task { await ... }`  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **Отписывайся** в `viewDidDisappear` / `viewWillDisappear` от уведомлений, KVO, делегатов  
- **Проверяй isViewLoaded / isBeingPresented** — иногда нужно различать первый показ и возврат  
- **Swift 6 strict concurrency** — все UI-обновления в `@MainActor`, сетевые вызовы — в `Task`  
- **Аналитика** — вызывай именно здесь (экран реально виден пользователю)  
- **Документируйте** — пиши комментарий «viewDidAppear — запуск загрузки данных при каждом показе экрана»

**Короткий девиз 2026**:
> «viewDidAppear — это когда экран уже **реально виден** пользователю и можно безопасно начинать работу: грузить данные, запускать анимации, трекать аналитику.  
> Делай здесь всё, что должно происходить при **каждом** появлении экрана, а не только при первом.  
> Всегда вызывай super в начале и не делай ничего тяжёлого.»

Удачи с правильным жизненным циклом и плавными экранами в Swift! 📱