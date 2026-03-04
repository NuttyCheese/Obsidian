**`traitCollectionDidChange(_:)`** — это метод жизненного цикла [[UITraitEnvironment]] (реализуется [[UIView]], [[UIViewController]], [[UIPresentationController]] и некоторыми другими классами), который вызывается **каждый раз**, когда меняется **коллекция трейтов** (trait collection) объекта.

С 2017 года (iOS 10+) и особенно после появления **Dynamic Type**, **тёмной темы**, **Split View**, **[[SceneDelegate]]** и **multi-window** в iPadOS это один из **самых важных** методов для адаптивного UI.

### Когда именно вызывается traitCollectionDidChange

| Событие / Изменение                              | Вызывается traitCollectionDidChange? | Что обычно изменилось в traitCollection |
|--------------------------------------------------|---------------------------------------|------------------------------------------|
| Пользователь изменил размер текста (Dynamic Type) | Да                                    | `preferredContentSizeCategory`           |
| Переключение между Light / Dark mode             | Да                                    | `userInterfaceStyle`                     |
| Поворот устройства (портрет ↔ ландшафт)          | Да (часто)                            | `horizontalSizeClass`, `verticalSizeClass` |
| Изменение размера окна / Split View / Slide Over | Да                                    | `horizontalSizeClass`, `verticalSizeClass`, `displayScale` |
| Подключение/отключение внешнего экрана           | Да                                    | `displayGamut`, `displayScale`           |
| Изменение accessibility-инверсии цвета           | Да                                    | `accessibilityContrast`                  |
| Смена языка / региона (очень редко)              | Да                                    | `layoutDirection`, `locale`              |

### Самые важные и часто используемые сценарии в 2026 году

#### 1. Реакция на смену тёмной/светлой темы (самый частый случай)

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    // Проверяем, изменился ли userInterfaceStyle
    if traitCollection.userInterfaceStyle != previousTraitCollection?.userInterfaceStyle {
        updateAppearanceForCurrentStyle()
    }
}

private func updateAppearanceForCurrentStyle() {
    switch traitCollection.userInterfaceStyle {
    case .dark:
        view.backgroundColor = .systemBackground
        // тёмная тема: тёмные цвета, иконки и т.д.
    case .light, .unspecified:
        view.backgroundColor = .systemGroupedBackground
        // светлая тема
    @unknown default:
        break
    }
}
```

#### 2. Адаптация под размер класса (size class) — iPad Split View / Slide Over

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    let oldHorizontal = previousTraitCollection?.horizontalSizeClass
    let newHorizontal = traitCollection.horizontalSizeClass
    
    if oldHorizontal != newHorizontal {
        updateLayoutForSizeClass()
    }
}

private func updateLayoutForSizeClass() {
    switch traitCollection.horizontalSizeClass {
    case .compact:
        // узкий экран → стек вертикальный, кнопки мелкие
        stackView.axis = .vertical
    case .regular:
        // широкий экран → стек горизонтальный, больше контента
        stackView.axis = .horizontal
    case .unspecified:
        break
    @unknown default:
        break
    }
}
```

#### 3. Адаптация шрифтов под Dynamic Type

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    if traitCollection.preferredContentSizeCategory != previousTraitCollection?.preferredContentSizeCategory {
        updateFontsForCurrentContentSize()
    }
}

private func updateFontsForCurrentContentSize() {
    titleLabel.font = .preferredFont(forTextStyle: .largeTitle)
    bodyLabel.font = .preferredFont(forTextStyle: .body)
    // все UILabel / UIButton автоматически обновятся, если использовать preferredFont
}
```

### Лучшие практики traitCollectionDidChange в Swift 2026

- **Всегда** вызывайте `super.traitCollectionDidChange(previousTraitCollection)` — иначе дочерние контроллеры/вью могут не обновиться  
- **Сравнивайте** с `previousTraitCollection` — это позволяет понять, **что именно изменилось** (userInterfaceStyle, sizeClass, contentSizeCategory)  
- **Не делайте** тяжёлые операции (сеть, диск, сложные вычисления) — метод может вызываться очень часто  
- **Для [[SwiftUI]]** — аналог — `.onChange(of: \.colorScheme)`, `.onChange(of: \.dynamicTypeSize)`, `.environment(\.horizontalSizeClass)`  
- **Для автоматической адаптации** — используйте `traitCollection` + `UITraitCollection.current` + `preferredFont(forTextStyle:)`  
- **В iPadOS** — обязательно тестируйте в Split View и Slide Over — traitCollection меняется динамически  
- **Документируйте** — пишите комментарий «traitCollectionDidChange — реакция на смену тёмной темы / size class / Dynamic Type»

**Короткий итог 2026**:
> `traitCollectionDidChange(_:)` — это **метод адаптации** интерфейса при изменении трейтов (тема, размер текста, size class, ориентация и т.д.).  
> В 2026 году:  
> - вызывайте `super`  
> - сравнивайте с `previousTraitCollection`  
> - используйте для: тёмной/светлой темы, Dynamic Type, адаптации под iPad Split View  
> - это **один из самых важных** методов для создания современного, адаптивного и доступного UI в UIKit  
