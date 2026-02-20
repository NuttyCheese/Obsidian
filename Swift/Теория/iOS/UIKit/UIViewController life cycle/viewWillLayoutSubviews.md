**`viewWillLayoutSubviews()`** — это метод жизненного цикла `UIViewController` в UIKit, который вызывается **непосредственно перед** тем, как система начнёт пересчитывать и применять layout (расположение и размеры) всех subviews контроллера.

Он входит в группу методов, связанных с **автоматическим layout** и **изменением размеров**.

### Когда вызывается viewWillLayoutSubviews()

| Событие / Причина изменения размеров                              | Вызывается viewWillLayoutSubviews()? | Примерный порядок вызова |
|-------------------------------------------------------------------|---------------------------------------|---------------------------|
| Изменение размеров контроллера (поворот, Split View, Slide Over) | Да                                    | → viewWillLayoutSubviews → layoutSubviews → viewDidLayoutSubviews |
| Изменение traitCollection (size class, Dynamic Type)              | Да (часто)                            | → traitCollectionDidChange → viewWillLayoutSubviews → ... |
| Вызов `setNeedsLayout()` / `layoutIfNeeded()` вручную             | Да                                    | → viewWillLayoutSubviews → layoutSubviews |
| Добавление/удаление subviews (addSubview, removeFromSuperview)    | Да (если изменились constraints)      | → viewWillLayoutSubviews → layoutSubviews |
| Изменение safeAreaInsets (notch, Dynamic Island, внешняя клавиатура) | Да                                    | → viewWillLayoutSubviews → layoutSubviews |
| Первый показ контроллера (после viewDidLoad)                      | Да                                    | → viewDidLoad → viewWillAppear → viewWillLayoutSubviews → ... |

### Самые популярные и рекомендуемые сценарии использования в 2026 году

#### 1. Подготовка к изменению layout (самый частый правильный кейс)

```swift
override func viewWillLayoutSubviews() {
    super.viewWillLayoutSubviews()
    
    // Здесь можно подготовить данные/свойства, которые нужны layoutSubviews
    // Например, пересчитать высоту header в зависимости от текущего размера
    if let headerView = tableView.tableHeaderView as? ProfileHeaderView {
        let targetWidth = view.bounds.width - 32 // с отступами
        headerView.preferredWidth = targetWidth
    }
}
```

#### 2. Корректировка constraints перед layout (очень частый паттерн)

```swift
private var bottomConstraint: NSLayoutConstraint?

override func viewWillLayoutSubviews() {
    super.viewWillLayoutSubviews()
    
    // Динамически меняем отступ снизу в зависимости от клавиатуры / safe area
    let bottomInset = view.safeAreaInsets.bottom + 16
    bottomConstraint?.constant = -bottomInset
}
```

#### 3. Предотвращение лишних layout-вызовов (оптимизация)

```swift
private var needsCustomLayout = false

override func viewWillLayoutSubviews() {
    super.viewWillLayoutSubviews()
    
    if needsCustomLayout {
        // выполняем кастомный layout только один раз
        performCustomLayout()
        needsCustomLayout = false
    }
}
```

### Чего **НИКОГДА** нельзя делать в viewWillLayoutSubviews() в 2026 году

| Запрещённое действие                              | Почему нельзя делать                                   | Куда перенести |
|---------------------------------------------------|--------------------------------------------------------|----------------|
| Изменение иерархии view (addSubview, removeFromSuperview) | Вызывает рекурсивный вызов layout → бесконечный цикл   | viewDidLoad / viewWillAppear |
| Вызов `setNeedsLayout()` / `layoutIfNeeded()`     | Рекурсия и бесконечный цикл                            | — |
| Тяжёлые вычисления, сетевые запросы               | Метод может вызываться очень часто (сотни раз за секунду при анимации) | viewDidLoad / viewDidAppear |
| Изменение navigationItem / tabBarItem             | Лучше делать в viewWillAppear / viewDidAppear          | viewWillAppear |
| Доступ к frame / bounds дочерних view             | Они ещё не пересчитаны (layout не выполнен)            | viewDidLayoutSubviews |

### Лучшие практики viewWillLayoutSubviews() в Swift 2026

- **Используйте** `viewWillLayoutSubviews()` **только** для:
  - подготовки данных/свойств, которые нужны для layoutSubviews
  - динамической корректировки constraints перед layout
  - лёгких расчётов, зависящих от текущих размеров view
- **Никогда** не вызывайте `setNeedsLayout()` внутри этого метода — это приведёт к рекурсии  
- **Всегда** вызывайте `super.viewWillLayoutSubviews()`  
- **Для финального расположения** — используйте `viewDidLayoutSubviews()`  
- **В SwiftUI** — аналог — `.onChange(of: geometry)` или `.background(GeometryReader)`  
- **Документируйте** — пишите комментарий «viewWillLayoutSubviews — подготовка constraints перед пересчётом layout (динамическая высота header)»

**Короткий итог 2026**:
> `viewWillLayoutSubviews()` — это **предупреждение** о том, что сейчас будет выполнен layout всех subviews контроллера.  
> В 2026 году:  
> - вызывается **перед** `layoutSubviews()`  
> - идеальное место для **подготовки** к layout (расчёт размеров, корректировка constraints)  
> - **запрещено** менять иерархию, вызывать setNeedsLayout, делать тяжёлые операции  
> - вызывайте `super`  
> Это **очень важный**, но **очень тонкий** метод — его неправильное использование → бесконечные циклы и тормоза.

Удачи с чистым и эффективным layout-циклом в твоём проекте! 📐