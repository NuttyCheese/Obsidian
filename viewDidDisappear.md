**viewDidDisappear(_:)** — это один из ключевых методов **жизненного цикла** [[UIViewController]] в [[UIKit]].

Он вызывается **каждый раз**, когда контроллер представления **перестаёт быть видимым** на экране (исчезает из иерархии окон).

### Когда именно вызывается viewDidDisappear (2026 актуально)

| Событие / Действие                            | viewDidDisappear вызывается?         | Примечание / важные детали                   |
| --------------------------------------------- | ------------------------------------ | -------------------------------------------- |
| Переход на другой экран (push / present)      | **Да**                               | После ухода текущего контроллера             |
| Возврат назад (pop / dismiss)                 | **Да** (на предыдущем экране)        | На том контроллере, который вернулся         |
| Переключение вкладки в [[UITabBarController]] | **Да** (на скрытой вкладке)          | При каждом переключении вкладки              |
| Показ модального контроллера сверху           | **Да** (на нижнем контроллере)       | Когда модальный экран закрывает текущий      |
| Приложение уходит в фон (background)          | **Да** (если контроллер был видимым) | UIApplication.didEnterBackgroundNotification |
| Контроллер удалён как child                   | **Да**                               | После removeFromParent()                     |
| Поворот экрана (orientation change)           | **Нет** (если не был скрыт)          | Обычно viewWillTransition(to:with:)          |

### Порядок вызовов (самая частая последовательность)

```text
1. viewWillDisappear(_:)
2. viewDidDisappear(_:)      ← здесь ты обычно
3. (следующий экран) viewWillAppear(_:)
4. (следующий экран) viewDidAppear(_:)
```

### Что обычно делают в viewDidDisappear в 2026 году

| Действие                                                         | Почему именно здесь (а не в viewWillDisappear)   | Пример кода (современный стиль)                    |
| ---------------------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| **Отписка от уведомлений** ([[NotificationCenter]])              | Экран уже не виден → нет смысла получать события | `NotificationCenter.default.removeObserver(self)`  |
| **Отписка от [[KVO]]** (observe)                                 | Предотвращает вызовы на невидимом контроллере    | `removeObserver(self, forKeyPath: "someKey")`      |
| **Остановка таймеров / анимаций / наблюдателей**                 | Экономия батареи и CPU                           | `timer.invalidate()` / `displayLink?.invalidate()` |
| **Остановка воспроизведения** ([[AVPlayer]], аудио)              | Чтобы не играть звук в фоне без необходимости    | `player?.pause()`                                  |
| **Сохранение состояния** (если не делал в [[viewWillDisappear]]) | Последний шанс перед полным исчезновением        | `saveScrollPosition()`                             |
| **Очистка ресурсов** (большие объекты, кэш)                      | Экран ушёл → можно безопасно освободить память   | `imageCache.removeAll()`                           |
| **Аналитика** (screen exit time)                                 | Точное время ухода с экрана                      | `Analytics.trackScreenExit("Profile")`             |

### Самый современный паттерн 2026 ([[async]]/[[await]] + [[@MainActor]])

```swift
@MainActor
class ProfileViewController: UIViewController {
    
    private var observationToken: Any?
    private var timer: Timer?
    private var player: AVPlayer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Подписка на уведомление (пример)
        observationToken = NotificationCenter.default.addObserver(
            forName: .userDidLogout,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleLogout()
        }
        
        timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
            print("Таймер тикает")
        }
        
        player = AVPlayer(url: videoURL)
    }
    
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        
        // 1. Отписка от уведомлений
        if let token = observationToken {
            NotificationCenter.default.removeObserver(token)
            observationToken = nil
        }
        
        // 2. Остановка таймера
        timer?.invalidate()
        timer = nil
        
        // 3. Остановка плеера
        player?.pause()
        player = nil
        
        // 4. Очистка кэша / больших объектов
        imageCache.removeAll()
        
        // 5. Аналитика ухода с экрана
        Analytics.trackScreenExit("Profile", duration: Date().timeIntervalSince(appearedTime))
    }
    
    private func handleLogout() {
        // логика выхода
    }
}
```

### Лучшие практики viewDidDisappear в Swift 2026

- **Всегда вызывай super.viewDidDisappear(animated)** — в начале метода  
- **Отписывайся от всего**, что подписался в [[viewDidAppear]] / [[viewWillAppear]]  
- **Не делай тяжёлую работу** — это не место для сетевых запросов или сложных вычислений  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **weak self** — в замыканиях, чтобы избежать [[retain cycle]]  
- **Swift 6 strict concurrency** — все UI-операции и отписки — в `@MainActor`  
- **Аналитика** — трекай именно здесь (экран реально исчез)  
- **Документируйте** — пиши комментарий «viewDidDisappear — отписка от уведомлений и остановка ресурсов»

**Короткий девиз 2026**:
> «viewDidDisappear — это когда экран уже **реально исчез** с экрана и пора срочно убирать за собой: отписываться, останавливать таймеры, плееры, наблюдатели и очищать ресурсы.  
> Делай здесь всё, что должно происходить при **каждом** уходе экрана, а не только при первом.  
> Всегда вызывай super в начале и не забывай отписываться от всего, что подписался.»
