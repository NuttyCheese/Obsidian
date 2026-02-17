**Size Classes** (размерные классы) в iOS — это механизм **адаптивного интерфейса**, который позволяет одному и тому же приложению выглядеть и вести себя по-разному в зависимости от:

- размера экрана устройства  
- ориентации (портрет/ландшафт)  
- режима многозадачности (Split View, Slide Over на iPad)  
- типа устройства (iPhone, iPad, Mac Catalyst, внешний дисплей)

Они были введены в **iOS 8** (2014) и до сих пор остаются одним из главных инструментов для создания **универсальных** (universal) интерфейсов в UIKit.

### 1. Какие бывают Size Classes (всего 4 комбинации)

Size Classes делят ширину и высоту экрана на два уровня:

| Ширина (Width) \ Высота (Height) | Compact (узкая/низкая)                  | Regular (широкая/высокая)               |
|-----------------------------------|------------------------------------------|------------------------------------------|
| **Compact Width**                 | iPhone в портрете                        | —                                        |
| **Regular Width**                 | iPhone в ландшафте, iPad в Split View    | iPad в полноэкранном режиме, Mac Catalyst |

Конкретные комбинации (wC-hC, wC-hR и т.д.):

| Комбинация       | Где встречается (2026 год)                                                                 | Типичный интерфейс |
|------------------|---------------------------------------------------------------------------------------------|---------------------|
| **wC hC**        | iPhone SE, iPhone 12 mini и т.д. в портрете (очень редко в ландшафте)                      | Всё в одну колонку, минимум элементов |
| **wC hR**        | Почти все современные iPhone в портрете                                                     | Одна колонка, вертикальный список, таб-бар снизу |
| **wR hC**        | iPhone в ландшафте, iPad в Slide Over / Split View (узкая часть)                           | Две колонки, боковая панель может прятаться |
| **wR hR**        | iPad полноэкранный, Mac Catalyst, внешний монитор                                          | Две/три колонки, боковая панель, много места |

**Запомнить просто**:
- **Compact Width** — почти всегда iPhone в портрете  
- **Regular Width** — iPad, iPhone в ландшафте, Mac  
- **Compact Height** — ландшафтный режим или узкая сцена  
- **Regular Height** — портретный режим на большинстве устройств

### 2. Для чего используют Size Classes (реальные сценарии 2026)

| Задача / Цель                                 | Как решают через Size Classes                                   | Где это видно в приложениях 2026 |
|-----------------------------------------------|------------------------------------------------------------------|-----------------------------------|
| Показать/скрыть боковую панель               | В wR → показываем sidebar, в wC → скрываем или делаем модальным | Почти все productivity-приложения (Notion, Things, Bear) |
| Перейти с одной колонки на две/три            | wC → список, wR → master-detail или grid                        | Mail, Notes, Files, Messages на iPad |
| Изменить расположение кнопок/панелей         | В hC → таб-бар снизу, в hR → сбоку                              | Музыкальные плееры, видео-приложения |
| Увеличить размер элементов / шрифтов          | В wR → крупные карточки, в wC → компактные                      | App Store, Podcasts, Apple Music |
| Поменять стиль навигации                      | В wC → таб-бар, в wR → sidebar + таб-бар                       | Twitter/X, Reddit, LinkedIn на iPad |
| Скрыть второстепенные элементы                | В wC → скрываем фильтры/поиск, в wR → показываем                | Таблицы данных, настройки |
| Изменить отступы и размеры шрифтов            | Вариации в traitCollection → разные констрейнты                 | Универсальные приложения (Medium, Bear) |

### 3. Как работают Size Classes на практике

Size Classes — это часть **trait collection** (`UITraitCollection`).

Каждый `UIViewController` и `UIView` имеет свойство:

```swift
var traitCollection: UITraitCollection { get }
```

Главные свойства, связанные с Size Classes:

```swift
traitCollection.horizontalSizeClass  // .compact или .regular
traitCollection.verticalSizeClass    // .compact или .regular
traitCollection.userInterfaceStyle   // .light / .dark
traitCollection.layoutDirection      // .leftToRight / .rightToLeft
```

### 4. Самые популярные способы использования Size Classes в 2026

#### Способ 1. Через Interface Builder (Storyboard/XIB)

Самый простой и до сих пор очень популярный:

1. В Interface Builder → включаешь **Vary for Traits** (кнопка внизу справа)  
2. Выбираешь комбинацию (wC-hR, wR-hR и т.д.)  
3. Меняешь констрейнты, скрываешь/показываешь элементы, меняешь шрифты только для этой комбинации  
4. Сохраняешь → изменения применяются автоматически на нужных устройствах

#### Способ 2. Программно в коде (самый гибкий)

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    let isRegularWidth = traitCollection.horizontalSizeClass == .regular
    
    if isRegularWidth {
        // iPad / Mac / iPhone ландшафт → показываем боковую панель
        sidebar.isHidden = false
        mainContent.leadingAnchor.constraint(equalTo: sidebar.trailingAnchor).isActive = true
    } else {
        // iPhone портрет → скрываем
        sidebar.isHidden = true
        mainContent.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
    }
    
    // Обновляем layout
    view.setNeedsLayout()
}
```

#### Способ 3. Через `override func viewWillTransition(to:with:)`

```swift
override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
    super.viewWillTransition(to: size, with: coordinator)
    
    coordinator.animate(alongsideTransition: { _ in
        if size.width > size.height {
            // Ландшафт → две колонки
            self.collectionView.collectionViewLayout = createTwoColumnLayout()
        } else {
            // Портрет → одна колонка
            self.collectionView.collectionViewLayout = createSingleColumnLayout()
        }
    })
}
```

### 5. Лучшие практики Size Classes в 2026

- **Не привязывайся жёстко к устройству** — проверяй `horizontalSizeClass` и `verticalSizeClass`, а не `UIDevice.current.userInterfaceIdiom`  
- **Используй safeAreaLayoutGuide** и `traitCollection` в связке — для адаптивных отступов  
- **Тестируй на всех комбинациях** — особенно wC-hR (iPhone портрет), wR-hC (iPad Slide Over), wR-hR (iPad полноэкранный)  
- **В SwiftUI** — аналог Size Classes встроен в `@Environment(\.horizontalSizeClass)` и `.horizontalSizeClass`  
- **Документируйте** — пиши комментарий «Адаптация layout для compact width (iPhone портрет)»

**Короткий девиз 2026**:
> Size Classes — это «умный способ сказать: на маленьком экране делай одну колонку, на большом — две, и не проверяй модель iPhone/iPad».  
> В 2026 году это **основной** инструмент адаптивного дизайна в UIKit.  
> Проверяй `horizontalSizeClass == .regular` → показывай больше контента, `== .compact` → упрощай интерфейс.

Удачи с универсальными и красивыми интерфейсами на всех устройствах! 📱