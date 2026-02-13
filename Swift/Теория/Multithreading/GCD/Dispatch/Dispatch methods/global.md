**Global queues** в **GCD** (Grand Central Dispatch) — это системные **конкурентные** (concurrent) очереди, которые Apple предоставляет "из коробки" для всех приложений. Они глобальные (shared), не нужно их создавать вручную, и они оптимизированы под [[QoS]] (Quality of Service).

На 2026 год ([[iOS]] 19/26, [[Swift]] 6+, Dispatch framework) ничего кардинально не изменилось в базовой модели global queues, но:

- QoS теперь ещё сильнее влияет на приоритет и энергопотребление (особенно на Apple Silicon и будущих чипах)
- Рекомендуется **избегать** частого использования `DispatchQueue.global()` без QoS для длительных задач (может привести к thread explosion и замедлению)
- Лучше использовать **явный QoS** или кастомные очереди с target queue hierarchy

### Основные global queues (DispatchQueue.global(qos: ...))

| QoS уровень              | Когда использовать                                      | Приоритет | Примеры задач                                      | Код (Swift) пример                                      |
|--------------------------|----------------------------------------------------------|-----------|----------------------------------------------------|---------------------------------------------------------|
| `.userInteractive`       | Всё, что влияет на отзывчивость UI прямо сейчас         | Очень высокий | Анимации, скролл, обработка тапов, touch events   | `DispatchQueue.global(qos: .userInteractive).async { ... }` |
| `.userInitiated`         | Задачи, инициированные пользователем, ждём результат быстро | Высокий   | Загрузка данных после нажатия кнопки, поиск, открытие экрана | `DispatchQueue.global(qos: .userInitiated).async { ... }` |
| `.utility` (default в Swift 3+) | Долгие задачи, где пользователю можно показать прогресс | Средний   | Скачивание файлов, обработка изображений, индексация | `DispatchQueue.global(qos: .utility).async { ... }`     |
| `.background`            | Задачи, которые не видны пользователю и не срочные      | Низкий    | Бэкап, синхронизация в фоне, аналитика, очистка кэша | `DispatchQueue.global(qos: .background).async { ... }`  |
| `.default`               | Старый стиль (до QoS), сейчас ≈ .utility                | Средний   | Совместимость со старым кодом                     | `DispatchQueue.global().async { ... }`                  |

**Важно**: `DispatchQueue.global()` без параметра QoS эквивалентно `.default` (≈ `.utility`).

### Лучшие практики на 2026 год

- **Не злоупотребляй `DispatchQueue.global()`** для длительных задач — это может привести к созданию лишних потоков и замедлению всего приложения ([[GCD]] под капотом управляет thread pool'ом глобально).
- Предпочитай:
  - Кастомные **serial queues** для защиты mutable state (actor-like поведение без actors)
  - Кастомные **concurrent queues** с target = global queue нужного QoS
  - **[[Swift Concurrency]]** ([[async]]/[[await]], [[Task]], [[actor]]) для нового кода — GCD остаётся для низкоуровневого контроля или legacy
- Для UI всегда возвращайся на main queue:

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // Тяжёлая работа: сеть, обработка фото и т.д.
    let result = heavyComputation()
    
    DispatchQueue.main.async {
        // Обновление UIKit/UI
        self.label.text = result
        self.tableView.reloadData()
    }
}
```

Или современный вариант (Swift 5.5+):

```swift
Task {
    let result = await heavyAsyncWork()  // или с GCD внутри
    
    await MainActor.run {
        self.label.text = result
    }
}
```

### Краткий вывод

**Global queues в GCD** — это быстрый способ запустить задачу в фоне без создания своей очереди.  
Самые частые: `.userInitiated` (для "почти UI") и `.utility` / `.background` (для настоящего фона).  
`DispatchQueue.global()` без QoS — ок для мелких быстрых задач, но для всего остального лучше указывать QoS явно.
