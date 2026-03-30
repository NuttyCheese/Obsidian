**`safeAreaLayoutGuide`** — это свойство в [[UIKit]], которое возвращает объект типа **`UILayoutGuide`**, представляющий **безопасную область** (safe area) внутри представления (`UIView`).

Safe area — это часть экрана, которая **гарантированно видна пользователю** и не перекрывается системными элементами интерфейса [[iOS]], такими как:

- notch / Dynamic Island (iPhone X и новее)
- статус-бар
- домашняя индикатор (home indicator)
- панель навигации / таб-бар (при их наличии)
- клавиатура (в некоторых случаях)

### Зачем нужен safeAreaLayoutGuide

До iOS 11 для отступов от краёв экрана использовали `topLayoutGuide` / `bottomLayoutGuide` (устарели) или фиксированные константы.  
С iOS 11+ Apple ввела `safeAreaLayoutGuide`, чтобы автоматически учитывать все системные наложения и изменения в разных устройствах и ориентациях.

### Основные свойства и якоря safeAreaLayoutGuide

```swift
view.safeAreaLayoutGuide.topAnchor
view.safeAreaLayoutGuide.bottomAnchor
view.safeAreaLayoutGuide.leadingAnchor
view.safeAreaLayoutGuide.trailingAnchor
view.safeAreaLayoutGuide.widthAnchor
view.safeAreaLayoutGuide.heightAnchor
view.safeAreaLayoutGuide.centerXAnchor
view.safeAreaLayoutGuide.centerYAnchor
```

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Стандартные отступы от краёв экрана (самый частый случай)

```swift
contentView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
    contentView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 16),
    contentView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -16),
    contentView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -16)
])
```

Это **золотой стандарт** для большинства экранов в UIKit-приложениях 2026 года.

#### 2. Центрирование контента внутри safe area

```swift
button.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    button.centerXAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerXAnchor),
    button.centerYAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerYAnchor),
    button.widthAnchor.constraint(equalToConstant: 200),
    button.heightAnchor.constraint(equalToConstant: 44)
])
```

#### 3. Фиксированная высота + отступ от верха и низа safe area

```swift
headerView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    headerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    headerView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    headerView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    headerView.heightAnchor.constraint(equalToConstant: 100)
])
```

#### 4. Scroll View + content inset (очень часто в 2026)

```swift
scrollView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    scrollView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
])

// Важно: для контента внутри scroll view
contentView.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: scrollView.contentLayoutGuide.topAnchor),
    contentView.leadingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.trailingAnchor),
    contentView.bottomAnchor.constraint(equalTo: scrollView.contentLayoutGuide.bottomAnchor),
    contentView.widthAnchor.constraint(equalTo: scrollView.frameLayoutGuide.widthAnchor)
])
```

### 5. Лучшие практики safeAreaLayoutGuide в 2026

- **Всегда** используй `safeAreaLayoutGuide` вместо прямого `superview` для top/bottom/leading/trailing  
- **Никогда** не используй фиксированные отступы от `topLayoutGuide` / `bottomLayoutGuide` — они устарели с iOS 11  
- **В iPhone с Dynamic Island** — safe area автоматически учитывает вырез и индикатор  
- **В iPad / Split View** — safe area адаптируется под размер окна  
- **В [[SwiftUI]]** — аналог — `.safeAreaInset(edge:)` или `.ignoresSafeArea()`  
- **Не меняй** константы внутри `layoutSubviews()` — используй `traitCollectionDidChange` или `viewDidLayoutSubviews` для динамических изменений  
- **Документируйте** — пиши комментарий «top = safeArea.top + 16 — стандартный отступ от Dynamic Island / статус-бара»

**Короткий девиз 2026**:
> `safeAreaLayoutGuide` — это «умные края экрана», которые всегда учитывают notch, Dynamic Island, статус-бар, home indicator и клавиатуру.  
> В 2026 году:  
> - используй **только safeAreaLayoutGuide** для всех отступов от краёв  
> - забудь про старые `topLayoutGuide` / `bottomLayoutGuide`  
> - комбинируй с `layoutMarginsGuide` для внутренних отступов  
> - это **обязательный** стандарт для любого современного UIKit-приложения
