**UIActivityViewController** — это системный контроллер в UIKit, который показывает стандартный iOS-шер (поделиться), позволяя пользователю отправить контент через любые доступные приложения и сервисы: Сообщения, Почта, AirDrop, WhatsApp, Telegram, Сохранить в Фото, Заметки, Напоминания, PDF в Файлы и т.д.

Это один из самых часто используемых контроллеров в iOS-приложениях, потому что он:
- выглядит **нативно** (как в системных приложениях Apple)
- автоматически обновляется с новыми сервисами и функциями iOS
- поддерживает **много типов контента** без лишнего кода
- обрабатывает **приватность** и разрешения сам

В 2026 году это **основной** способ поделиться чем-либо в UIKit-приложениях.

### Основные возможности (актуально на iOS 19 / 2026)

| Возможность                              | Поддержка в 2026 | Пример контента |
|------------------------------------------|------------------|-----------------|
| Текст (String)                           | Полная           | Сообщение, ссылка, цитата |
| URL (веб-ссылка)                         | Полная           | Сайт, YouTube, статья |
| Изображение (UIImage)                    | Полная           | Фото, скриншот, мем |
| PDF / документ (Data, URL)               | Полная           | Счёт, договор, экспорт |
| Несколько элементов (items: [Any])       | Полная           | Фото + текст + ссылка |
| Кастомные действия (activityItemsSource) | Полная           | Свой сервис (например, "Поделиться в нашем чате") |
| Исключение сервисов (excludedActivityTypes) | Полная        | Скрыть AirDrop, Сохранить в Фото |
| Предпросмотр (activityViewControllerLinkMetadata) | Полная (LPLinkMetadata) | Красивый предпросмотр ссылки |
| Поддержка SharePlay / Share Sheet        | Полная           | Совместный просмотр видео/музыки |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Поделиться текстом + ссылкой + изображением

```swift
final class ShareManager {
    
    static func presentShareSheet(from viewController: UIViewController,
                                 text: String?,
                                 url: URL?,
                                 image: UIImage?,
                                 sourceView: UIView? = nil) {
        
        var items: [Any] = []
        
        if let text { items.append(text) }
        if let url { items.append(url) }
        if let image { items.append(image) }
        
        guard !items.isEmpty else { return }
        
        let activityVC = UIActivityViewController(activityItems: items, applicationActivities: nil)
        
        // Исключаем ненужные сервисы (опционально)
        activityVC.excludedActivityTypes = [
            .addToReadingList,
            .assignToContact,
            .print,
            .saveToCameraRoll
        ]
        
        // Красивый предпросмотр ссылки (если есть URL)
        if let url {
            let metadata = LPLinkMetadata()
            metadata.originalURL = url
            metadata.title = "Ссылка на статью"
            // можно добавить иконку, изображение и т.д.
            activityVC.linkMetadata = metadata
        }
        
        // Для iPad — popover
        if let popover = activityVC.popoverPresentationController,
           let source = sourceView {
            popover.sourceView = source
            popover.sourceRect = source.bounds
            popover.permittedArrowDirections = .any
        }
        
        viewController.present(activityVC, animated: true)
    }
}
```

Использование:

```swift
ShareManager.presentShareSheet(from: self,
                               text: "Смотри эту статью!",
                               url: URL(string: "https://example.com/article"),
                               image: UIImage(named: "preview"),
                               sourceView: shareButton)
```

#### 2. Поделиться только одной картинкой (классика)

```swift
@IBAction func shareImage(_ sender: UIButton) {
    guard let image = UIImage(named: "photo") else { return }
    
    let activityVC = UIActivityViewController(activityItems: [image], applicationActivities: nil)
    
    // Для iPad
    activityVC.popoverPresentationController?.sourceView = sender
    activityVC.popoverPresentationController?.sourceRect = sender.bounds
    
    present(activityVC, animated: true)
}
```

#### 3. Кастомное действие в Share Sheet

```swift
let customActivity = UIActivity()
customActivity.title = "Поделиться в нашем чате"
customActivity.image = UIImage(systemName: "message.circle.fill")

// Реализация perform(withActivityItems:)
class ChatShareActivity: UIActivity {
    override var activityType: UIActivity.ActivityType? { .init("com.myapp.shareToChat") }
    override var activityTitle: String? { "В наш чат" }
    override var activityImage: UIImage? { UIImage(systemName: "message.circle.fill") }
    
    override func canPerform(withActivityItems activityItems: [Any]) -> Bool {
        activityItems.contains { $0 is String || $0 is URL || $0 is UIImage }
    }
    
    override func perform(withActivityItems activityItems: [Any]) {
        // Открываем чат и передаём контент
        print("Отправляем в чат: \(activityItems)")
    }
}

let activityVC = UIActivityViewController(activityItems: items,
                                         applicationActivities: [ChatShareActivity()])
```

### Лучшие практики UIActivityViewController в Swift 2026

- **Всегда** указывай `sourceView` / `sourceRect` для iPad — иначе краш на iPad  
- **excludedActivityTypes** — скрывай ненужные сервисы (например, `.print` в мобильных приложениях)  
- **LPLinkMetadata** — обязательно для ссылок: красивый предпросмотр с заголовком, иконкой, изображением  
- **Не передавай слишком много данных** — лучше один URL или одно изображение  
- **Проверяй доступность** — `UIActivityViewController.canPerform(withActivityItems:)`  
- **@MainActor** — всегда презентуй на главном потоке  
- **Swift 6 strict concurrency** — полностью безопасен  
- **Документируйте** — пиши комментарий «UIActivityViewController — стандартный шер с предпросмотром ссылки»

**Короткий девиз 2026**:
> UIActivityViewController — это **системный шер**, который выглядит как родной iOS, обновляется с каждой версией и поддерживает всё: текст, ссылки, фото, PDF, файлы.  
> В 2026 году это **единственный правильный** способ дать пользователю поделиться контентом.  
> Всегда добавляй `sourceView` для iPad, LPLinkMetadata для ссылок и исключай ненужные сервисы.

Удачи с удобным и красивым шарингом в твоём приложении! 📤