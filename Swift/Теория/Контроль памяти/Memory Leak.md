#memory_management #memory_leak #ios #swift #objective_c #retain_cycle #arc #debugging #performance #heap
# Утечки памяти (Memory Leaks) в [[Swift]] / [[iOS]]

**Утечка памяти** — ситуация, когда объект остаётся в памяти, хотя больше **не нужен** и на него **нет активных ссылок**, которые могли бы его использовать.  
В результате приложение постепенно потребляет всё больше памяти → замедление → краш (jetsam) или memory pressure.

### Основные причины утечек в Swift (2026)

| Причина                                    | Как возникает                                     | Как проявляется                     | Частота в iOS-приложениях |
| ------------------------------------------ | ------------------------------------------------- | ----------------------------------- | ------------------------- |
| **Цикл сильных ссылок** ([[retain cycle]]) | Два+ объекта сильно ссылаются друг на друга       | retain count никогда не = 0         | ★★★★★ (самая частая)      |
| **Замыкание захватывает self сильно**      | `{ self.doSomething() }` без `[weak self]`        | Контроллер/объект живёт вечно       | ★★★★☆                     |
| **Делегат / datasource сильный**           | `delegate` или `dataSource` — strong по умолчанию | Контроллер не умирает после dismiss | ★★★☆☆                     |
| **NSNotificationCenter без remove**        | Не отписались в `deinit`                          | Объект продолжает получать события  | ★★☆☆☆                     |
| **Timer / CADisplayLink сильный**          | `timer = Timer.scheduledTimer…` без `[weak self]` | Таймер держит объект вечно          | ★★☆☆☆                     |
| **Статические / глобальные ссылки**        | `static var cache = MyObject()`                   | Объект живёт до конца приложения    | ★☆☆☆☆                     |

### Как обнаружить утечки (основные инструменты)

| Инструмент                    | Что показывает                          | Когда использовать              | Как запускать                         |
| ----------------------------- | --------------------------------------- | ------------------------------- | ------------------------------------- |
| **Instruments → Leaks**       | Реальные утечки (объекты без ссылок)    | Всегда после тестирования       | [[Xcode]] → Product → Profile → Leaks |
| **Instruments → Allocations** | Рост памяти + живые объекты             | Поиск пиков и утечек            | Allocations template                  |
| **Memory Graph Debugger**     | Циклы сильных ссылок (граф объектов)    | Когда подозреваешь retain cycle | Debug → Memory Graph (иконка в Xcode) |
| **Xcode → View Memory Graph** | То же, но в реальном времени            | Быстрая проверка контроллеров   | Debug navigator → Memory              |
| **MLeaksFinder** (сторонний)  | Автоматически ловит утечки контроллеров | Быстрый тест навигации          | CocoaPods / SPM                       |

### Как предотвратить утечки (чек-лист 2026)

1. **Замыкания** — всегда `[weak self]` по умолчанию  
   ```swift
   someAsync { [weak self] in
       self?.updateUI()
   }
   ```

2. **Делегаты / datasource / observers** — делай `weak`  
   ```swift
   weak var delegate: SomeDelegate?
   NotificationCenter.default.removeObserver(self) в deinit
   ```

3. **Parent → Child** — сильная ссылка от родителя к ребёнку  
   **Child → Parent** — [[weak]] или [[unowned]]  
   ```swift
   class Child {
       weak var parent: Parent?
   }
   ```

4. **Timer / CADisplayLink / repeating closures**  
   Всегда `[weak self]` или invalidate в [[deinit]]

5. **Статические / глобальные объекты**  
   Используй `weak` или очищай вручную

6. **Проверяй в deinit**  
   ```swift
   deinit {
       print("🗑 \(type(of: self)) deinit")
   }
   ```
   Если `deinit` не вызывается — почти всегда [[retain cycle]].

### Короткий чек-лист перед релизом

- Все замыкания внутри классов → `[weak self]` или `[unowned self]`
- Все делегаты / datasource → `weak`
- Все NotificationCenter observers → remove в `deinit`
- Все таймеры / display link → invalidate в `deinit`
- Прогоняй **Memory Graph Debugger** после навигации по экранам
- Запусти **Leaks** в Instruments на 10–15 минут активного использования

**Золотое правило Swift 2026**:
> «Если объект живёт дольше, чем должен — 99% случаев это retain cycle через замыкание или делегат.  
> Пиши `[weak self]` по умолчанию — это бесплатно и спасает от 95% утечек.»
