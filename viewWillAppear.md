**viewWillAppear(_:)** — это один из ключевых методов **жизненного цикла** [[UIViewController]] в [[UIKit]].

Он вызывается **каждый раз**, когда контроллер представления **вот-вот станет видимым** на экране (перед анимацией появления).

### Когда именно вызывается viewWillAppear

| Событие / Действие                                 | viewWillAppear вызывается?           | Сколько раз за жизнь контроллера    | Примечание / важные детали                        |
| -------------------------------------------------- | ------------------------------------ | ----------------------------------- | ------------------------------------------------- |
| Первый показ экрана (push / present)               | **Да**                               | **Один раз** + каждый возврат       | Перед анимацией появления                         |
| Возврат назад (pop / dismiss)                      | **Да**                               | Каждый раз при возврате             | На том контроллере, который вернулся              |
| Переключение вкладки в [[UITabBarController]]      | **Да**                               | При каждом выборе вкладки           | Даже если контроллер уже создан                   |
| Показ модального контроллера сверху и его закрытие | **Да** (на нижнем контроллере)       | Каждый раз при возврате видимости   | После dismiss модального                          |
| Приложение вернулось из фона (foreground)          | **Да** (если контроллер был видимым) | Каждый раз при активации приложения | [[UIApplication]].willEnterForegroundNotification |
| Контроллер добавлен как child                      | **Да**                               | При первом появлении                | После addChild → didMove(toParent:)               |

### Порядок вызовов (самая частая последовательность)

```text
1. viewDidLoad()                          // один раз
2. viewWillAppear(_:)                     ← здесь ты обычно
3. viewDidAppear(_:)
4. (при уходе) viewWillDisappear(_:)
5. viewDidDisappear(_:)
```

### Что обычно делают в viewWillAppear в 2026 году

| Действие                                                      | Почему именно здесь (а не в [[viewDidLoad]] / [[viewDidAppear]]) | Пример кода (современный стиль)               |
| ------------------------------------------------------------- | ---------------------------------------------------------------- | --------------------------------------------- |
| **Начало асинхронной загрузки данных**                        | Экран ещё не виден → пользователь не видит задержку              | `Task { await refreshData() }`                |
| **Обновление UI перед показом**                               | Можно обновить данные до анимации появления                      | `updateUIWithLatestData()`                    |
| **Регистрация на уведомления / наблюдатели** (короткосрочные) | Подписка нужна только пока экран виден                           | `NotificationCenter.default.addObserver(...)` |
| **Аналитика** (screen view start)                             | Точное время начала показа экрана                                | `Analytics.trackScreenView("Profile")`        |
| **Сброс состояния** (scroll position, selection)              | При каждом появлении экрана сбрасывать к начальному виду         | `tableView.scrollToTop(animated: false)`      |
| **Проверка авторизации / [[deep link]]**                      | Перед показом — можно перенаправить на логин                     | `if !isAuthorized { showLogin() }`            |
| **Запуск анимаций / фокус**                                   | Перед появлением — подготовить к плавному входу                  | `textField.becomeFirstResponder()`            |

### Самый современный паттерн 2026 ([[@MainActor]] + [[async]]/[[await]])

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
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        // 1. Начать загрузку данных (пользователь ещё не видит экран)
        Task {
            await refreshData()
        }
        
        // 2. Обновить UI перед анимацией появления
        updateUIWithLatestData()
        
        // 3. Аналитика начала показа экрана
        Analytics.trackScreenView("Profile")
        
        // 4. Подписка на уведомления (если нужно только пока экран виден)
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(handleUserUpdate),
                                               name: .userProfileUpdated,
                                               object: nil)
    }
    
    private func refreshData() async {
        do {
            try await viewModel.loadProfile()
            updateUIWithLatestData()
        } catch {
            showError(error)
        }
    }
    
    private func updateUIWithLatestData() {
        // обновление UILabel, UITableView и т.д. на основе viewModel
        nameLabel.text = viewModel.user?.name
        // ...
    }
    
    @objc private func handleUserUpdate() {
        Task { await refreshData() }
    }
    
    // Важно! Отписка в viewDidDisappear / viewWillDisappear
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        NotificationCenter.default.removeObserver(self)
    }
}
```

### Лучшие практики viewWillAppear в Swift 2026

- **Всегда вызывай super.viewWillAppear(animated)** — в начале метода  
- **Начинай асинхронную работу здесь** — загрузка данных, аналитика, подписки (пользователь ещё не видит экран)  
- **Не делай тяжёлую синхронную работу** — максимум 1–2 мс (иначе задержка появления экрана)  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **Отписывайся** в `viewDidDisappear` / `viewWillDisappear` от уведомлений, [[KVO]], делегатов  
- **Swift 6 strict concurrency** — все UI-операции — в `@MainActor`, сетевые вызовы — в `Task`  
- **Документируйте** — пиши комментарий «viewWillAppear — начало загрузки данных и аналитики перед показом экрана»

**Короткий девиз 2026**:
> «viewWillAppear — это когда экран **вот-вот появится**, и пора подготовиться: начать загрузку данных, зарегистрировать наблюдателей, трекать аналитику и сбросить состояние.  
> Делай здесь всё, что должно происходить **при каждом** появлении экрана, пока он ещё не виден пользователю.  
> Всегда вызывай super в начале и не блокируй UI тяжёлыми операциями.»
