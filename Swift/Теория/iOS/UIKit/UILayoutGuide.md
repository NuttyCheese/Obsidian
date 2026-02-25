**UILayoutGuide** — это лёгкий, невидимый объект в Auto Layout, который используется как **виртуальная направляющая** (reference) для создания и управления ограничениями (constraints), без необходимости добавлять реальный `UIView`.

С помощью `UILayoutGuide` вы получаете все преимущества системы layout (автоматическое обновление при изменении размеров, size class, trait collection), но **без накладных расходов** на настоящий view (рендеринг, hit-testing, иерархия).

### Когда использовать UILayoutGuide (самые частые сценарии 2026)

| Сценарий                                      | Почему именно UILayoutGuide (а не UIView)            | Альтернатива (когда UIView лучше) |
|-----------------------------------------------|-------------------------------------------------------|------------------------------------|
| Отступы от safe area / readable content       | Не нужен реальный view, только якоря                  | — |
| Разделение экрана на зоны (header / content / footer) | Чистый код, меньше слоёв в иерархии                   | — |
| Выравнивание нескольких элементов по центру/краю | Один guide вместо dummy view                          | — |
| Кастомные отступы между группами элементов    | Легко менять константы в коде                         | — |
| Создание «невидимого» контейнера для constraints | Производительность лучше, чем у прозрачного UIView    | — |
| Адаптация layout под разные size class / ориентацию | Полностью совместим с traitCollectionDidChange        | — |

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
class ProfileViewController: UIViewController {
    
    private let scrollView = UIScrollView()
    private let contentView = UIView()
    
    // Направляющие — невидимые, но очень полезные
    private let readableContentGuide = UILayoutGuide()
    private let topContentGuide = UILayoutGuide()
    private let bottomContentGuide = UILayoutGuide()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupHierarchy()
        setupGuides()
        setupConstraints()
    }
    
    private func setupHierarchy() {
        view.backgroundColor = .systemBackground
        
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        contentView.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(scrollView)
        scrollView.addSubview(contentView)
        
        // Добавляем направляющие в иерархию view (важно!)
        view.addLayoutGuide(readableContentGuide)
        view.addLayoutGuide(topContentGuide)
        view.addLayoutGuide(bottomContentGuide)
    }
    
    private func setupGuides() {
        // 1. Readable content guide — адаптируется под большие экраны (iPad, landscape)
        readableContentGuide.widthAnchor.constraint(equalTo: view.readableContentGuide.widthAnchor).isActive = true
        readableContentGuide.centerXAnchor.constraint(equalTo: view.readableContentGuide.centerXAnchor).isActive = true
        
        // 2. Верхний и нижний отступы (можно менять динамически)
        topContentGuide.heightAnchor.constraint(equalToConstant: 20).isActive = true
        bottomContentGuide.heightAnchor.constraint(equalToConstant: 40).isActive = true
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // ScrollView занимает весь экран
            scrollView.topAnchor.constraint(equalTo: view.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            
            // ContentView внутри scrollView
            contentView.topAnchor.constraint(equalTo: scrollView.contentLayoutGuide.topAnchor),
            contentView.bottomAnchor.constraint(equalTo: scrollView.contentLayoutGuide.bottomAnchor),
            contentView.leadingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.leadingAnchor),
            contentView.trailingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.trailingAnchor),
            contentView.widthAnchor.constraint(equalTo: scrollView.frameLayoutGuide.widthAnchor),
            
            // Используем readableContentGuide для контента
            // (например, аватарка и текст по центру с отступами)
            // ...
        ])
    }
    
    // Динамическая адаптация при изменении trait collection
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        if traitCollection.horizontalSizeClass == .regular {
            // На iPad или в landscape делаем больше отступов
            topContentGuide.heightAnchor.constant = 60
            bottomContentGuide.heightAnchor.constant = 80
        } else {
            topContentGuide.heightAnchor.constant = 20
            bottomContentGuide.heightAnchor.constant = 40
        }
    }
}
```

### Сравнение UILayoutGuide vs UIView (dummy view)

| Критерий                                  | UILayoutGuide                                      | UIView (прозрачный dummy view)                   | Победитель в 2026 |
|-------------------------------------------|-----------------------------------------------------|--------------------------------------------------|-------------------|
| Накладные расходы на рендеринг            | Нет                                                | Есть (даже если hidden и alpha = 0)              | UILayoutGuide     |
| Влияние на иерархию view                  | Не добавляется в subviews                           | Добавляется → hit-test, accessibility            | UILayoutGuide     |
| Поддержка Auto Layout                     | Полная                                             | Полная                                           | —                 |
| Легко отлаживать в View Debugger         | Да (виден как guide)                               | Да, но загромождает иерархию                     | UILayoutGuide     |
| Можно анимировать                         | Да (через constraints)                             | Да                                               | —                 |
| Можно добавлять gesture recognizer        | Нет                                                | Да                                               | UIView            |

**Вывод**: используйте **UILayoutGuide** в 95% случаев, когда вам нужен только «невидимый якорь» для constraints.  
UIView — только если нужен реальный view (hit-test, жесты, доступность).

### Лучшие практики UILayoutGuide в 2026 году

- **Всегда** добавляйте guide в иерархию через `view.addLayoutGuide(_:)`  
- **Используйте** `readableContentGuide` — это встроенная направляющая для читаемого контента (особенно на iPad)  
- **Для динамических отступов** — меняйте константы в `traitCollectionDidChange`  
- **Для scroll view** — используйте `contentLayoutGuide` и `frameLayoutGuide` вместо ручного `contentSize`  
- **Для SwiftUI** — аналогов нет напрямую, но эквивалент — `GeometryReader`, `Spacer`, `alignmentGuide`  
- **Документируйте** — пишите комментарий:

```swift
/// Направляющая для центрирования контента с учётом readable margins (адаптивно под iPad)
private let readableContentGuide = UILayoutGuide()
```

**Короткий итог 2026**:
> `UILayoutGuide` — **невидимая направляющая** для Auto Layout, которая заменяет dummy `UIView` без лишних затрат.  
> В 2026 году:  
> - ключевые методы — `addLayoutGuide`, `frameOfPresentedViewInContainerView` (для кастомных модалок)  
> - идеален для отступов, центрирования, зон контента, адаптации под size class  
> - производительнее и чище, чем прозрачный UIView  
> - это **must-have** инструмент для любого современного UIKit-layout  

Удачи с чистыми, производительными и адаптивными интерфейсами в твоём проекте! 📐