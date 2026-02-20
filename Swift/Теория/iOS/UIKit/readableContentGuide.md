**readableContentGuide** — это свойство [[UIView]] в [[UIKit]], которое представляет **рекомендуемую область** для размещения **читаемого контента** (текста, полей ввода, кнопок и т.д.) внутри контроллера представления.

С iOS 9 (2015) и по 2026 год это **один из самых важных** механизмов для создания адаптивного и удобного интерфейса, особенно на iPad и iPhone с разными размерами экрана и в режиме Split View / Slide Over.

### Что именно представляет readableContentGuide

```swift
var readableContentGuide: UILayoutGuide { get }
```

Это **[[UILayoutGuide]]**, который автоматически:
- учитывает **безопасные области** (safe area),
- добавляет **рекомендуемые отступы** для читаемости текста,
- адаптируется к размеру экрана, ориентации и режиму многозадачности (Split View на iPad),
- имеет ширину, близкую к **читабельной** (примерно 672–780 pt на iPad, меньше на iPhone).

**Важно**:  
`readableContentGuide` **не равен** `safeAreaLayoutGuide`  
`readableContentGuide` **уже** и **центрирован** внутри safe area.

### Типичные значения ширины readableContentGuide (2026, примерные)

| Устройство / режим                | Примерная ширина readableContentGuide | Когда видно |
|-----------------------------------|---------------------------------------|-------------|
| iPhone (портрет)                  | ~320–380 pt                           | Почти всегда |
| iPhone (ландшафт)                 | ~500–600 pt                           | На больших моделях |
| iPad (Split View 1/3 экрана)      | ~320 pt                               | iPadOS многозадачность |
| iPad (Split View 1/2 экрана)      | ~500–600 pt                           | iPadOS многозадачность |
| iPad (полноэкранный)              | 672–780 pt                            | Классический «читабельный» размер |

### Самые популярные и рекомендуемые паттерны использования (2026)

#### 1. Центрирование контента с отступами для читаемости (самый частый)

```swift
class ArticleViewController: UIViewController {
    
    private let scrollView = UIScrollView()
    private let contentStack = UIStackView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(scrollView)
        
        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: view.topAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        contentStack.axis = .vertical
        contentStack.spacing = 24
        contentStack.translatesAutoresizingMaskIntoConstraints = false
        scrollView.addSubview(contentStack)
        
        // Самое важное — привязываем к readableContentGuide
        NSLayoutConstraint.activate([
            contentStack.topAnchor.constraint(equalTo: scrollView.topAnchor, constant: 32),
            contentStack.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor, constant: -32),
            contentStack.leadingAnchor.constraint(equalTo: readableContentGuide.leadingAnchor),
            contentStack.trailingAnchor.constraint(equalTo: readableContentGuide.trailingAnchor),
            contentStack.widthAnchor.constraint(equalTo: readableContentGuide.widthAnchor)
        ])
        
        // Добавляем текст и элементы
        let titleLabel = UILabel()
        titleLabel.text = "Длинная статья"
        titleLabel.font = .preferredFont(forTextStyle: .largeTitle)
        titleLabel.numberOfLines = 0
        contentStack.addArrangedSubview(titleLabel)
        
        // ... остальные элементы
    }
}
```

#### 2. Автоматическое центрирование формы / карточки

```swift
let cardView = UIView()
cardView.backgroundColor = .systemBackground
cardView.layer.cornerRadius = 16
cardView.layer.shadowOpacity = 0.1
cardView.layer.shadowRadius = 8

cardView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(cardView)

NSLayoutConstraint.activate([
    cardView.centerXAnchor.constraint(equalTo: readableContentGuide.centerXAnchor),
    cardView.centerYAnchor.constraint(equalTo: readableContentGuide.centerYAnchor),
    cardView.widthAnchor.constraint(equalTo: readableContentGuide.widthAnchor, multiplier: 0.9),
    cardView.heightAnchor.constraint(equalToConstant: 400)
])
```

#### 3. Адаптивная ширина текста / web-контента

```swift
webView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(webView)

NSLayoutConstraint.activate([
    webView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    webView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),
    webView.leadingAnchor.constraint(equalTo: readableContentGuide.leadingAnchor),
    webView.trailingAnchor.constraint(equalTo: readableContentGuide.trailingAnchor)
])
```

### Лучшие практики readableContentGuide в Swift 2026

- **Всегда** используйте `readableContentGuide` для **текстового контента** (статей, форм, настроек, описаний)  
- **Не путайте** с `safeAreaLayoutGuide` — readableContentGuide **уже** и **центрирован**  
- **Для iPad** — это критически важно: без него текст становится слишком широким и нечитаемым в Split View  
- **Для iPhone** — полезно при ландшафтном режиме и на больших экранах (Pro Max)  
- **В [[SwiftUI]]** — аналог — `.frame(maxWidth: .readableContent)` или `.containerRelativeFrame` с `.horizontal`  
- **Документируйте** — пишите комментарий «readableContentGuide — адаптивная ширина для читаемого контента (672–780 pt на iPad)»

**Короткий итог 2026**:
> `readableContentGuide` — это **рекомендуемая ширина** для размещения текста и форм, чтобы они были удобочитаемыми на всех устройствах и режимах.  
> В 2026 году:  
> - привязывайте ведущие/трейлинговые якоря контента к `readableContentGuide.leading` / `.trailing`  
> - ширина автоматически адаптируется: узкая на iPhone, оптимальная на iPad  
> - это **один из главных** инструментов создания **адаптивного** и **читабельного** интерфейса в UIKit  
