**UIDocumentInteractionController** — это класс в [[UIKit]], который предоставляет **системный интерфейс** для взаимодействия с файлами: просмотр, открытие в других приложениях, печать, отправка по AirDrop, сообщение, почта, сохранение в «Файлы» и т.д.

Это **стандартный** и **самый рекомендуемый** способ в UIKit показать пользователю меню «Открыть в…», «Поделиться» или «Быстрый просмотр» для любого файла ([[PDF]], изображение, документ, архив и т.д.).

### Когда использовать UIDocumentInteractionController в 2026 году

| Сценарий                                     | Почему именно UIDocumentInteractionController             | Альтернатива в 2026                                   |
| -------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------- |
| Показать файл из приложения (PDF, DOCX, JPG) | Нативный системный UI, поддержка всех приложений          | [[UIActivityViewController]] (если только поделиться) |
| Поддержка «Открыть в…» (Open In)             | Единственный официальный способ для Open In               | —                                                     |
| Быстрый просмотр файла (Quick Look)          | Встроенный предпросмотр без открытия сторонних приложений | QLPreviewController (если нужен только просмотр)      |
| AirDrop, Сообщения, Почта, Сохранить в Файлы | Всё в одном месте, единый UX                              | UIActivityViewController + custom options             |
| Работа с файлами из «Файлы» / iCloud         | Полная интеграция с системным файловым менеджером         | —                                                     |

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
import UIKit

class DocumentViewerViewController: UIViewController {
    
    private var documentURL: URL?
    private var documentInteractionController: UIDocumentInteractionController?
    
    init(fileURL: URL) {
        self.documentURL = fileURL
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        guard let url = documentURL else { return }
        
        // 1. Создаём контроллер
        documentInteractionController = UIDocumentInteractionController(url: url)
        documentInteractionController?.delegate = self
        
        // 2. Показываем меню (самый частый сценарий)
        if documentInteractionController?.presentOpenInMenu(from: view.bounds, in: view, animated: true) == false {
            // Если Open In недоступен — показываем обычное меню
            documentInteractionController?.presentOptionsMenu(from: view.bounds, in: view, animated: true)
        }
    }
    
    // Альтернатива: показать Quick Look (предпросмотр)
    private func showQuickLook() {
        documentInteractionController?.presentPreview(animated: true)
    }
    
    // Альтернатива: показать все опции (Share, Print, AirDrop и т.д.)
    private func showOptionsMenu() {
        documentInteractionController?.presentOptionsMenu(from: view.bounds, in: view, animated: true)
    }
}

// MARK: - UIDocumentInteractionControllerDelegate
extension DocumentViewerViewController: UIDocumentInteractionControllerDelegate {
    
    // Предпросмотр (Quick Look)
    func documentInteractionControllerViewControllerForPreview(_ controller: UIDocumentInteractionController) -> UIViewController {
        return self
    }
    
    func documentInteractionControllerViewForPreview(_ controller: UIDocumentInteractionController) -> UIView? {
        return view  // или конкретный view, над которым показывать
    }
    
    func documentInteractionControllerRectForPreview(_ controller: UIDocumentInteractionController) -> CGRect {
        return view.bounds  // или конкретная область
    }
    
    // Опционально: обработка успешного открытия в другом приложении
    func documentInteractionControllerDidDismissOpenInMenu(_ controller: UIDocumentInteractionController) {
        print("Меню 'Открыть в…' закрыто")
    }
    
    func documentInteractionControllerDidDismissOptionsMenu(_ controller: UIDocumentInteractionController) {
        print("Меню опций закрыто")
    }
}
```

### Современные альтернативы UIDocumentInteractionController в 2026 году

| Альтернатива                              | Когда лучше UIDocumentInteractionController        | Минимальная версия | Плюсы |
|-------------------------------------------|-----------------------------------------------------|---------------------|-------|
| **UIActivityViewController**              | Нужно только поделиться / AirDrop / Сохранить в Файлы | iOS 13+             | Более гибкий, поддержка кастомных activity |
| **QLPreviewController**                   | Только предпросмотр (Quick Look) без меню           | iOS 13+             | Чистый просмотр документа |
| **UIDocumentPickerViewController**        | Импорт/экспорт файлов из «Файлы»                    | iOS 13+             | Полный доступ к файловой системе |
| **SwiftUI ShareLink / FileRepresentation** | Новый проект на SwiftUI                             | iOS 16+             | Декларативный стиль, проще интеграция |
| **LPLinkMetadata + UIActivityViewController** | Современный share sheet с превью (как в Сообщениях) | iOS 13+             | Красивый предпросмотр ссылки/файла |

### Лучшие практики UIDocumentInteractionController в 2026 году

- **Всегда** проверяйте результат `presentOpenInMenu` / `presentOptionsMenu` — если false, значит нет подходящих приложений  
- **Обязательно** реализуйте `UIDocumentInteractionControllerDelegate` — особенно методы для предпросмотра (`viewControllerForPreview`, `viewForPreview`)  
- **Для Quick Look** — используйте `presentPreview(animated:)` — это самый простой способ показать файл без открытия сторонних приложений  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint` на кнопке, которая вызывает контроллер  
- **Для SwiftUI** — используйте `ShareLink` или `FileRepresentation` — UIDocumentInteractionController нужен только в UIKit  
- **Для Combine / async** — запускайте в `Task` и обрабатывайте результат через замыкание  
- **Документируйте** — пишите комментарий:

```swift
/// Показывает системное меню 'Открыть в…' / 'Поделиться' для файла
func presentDocumentInteraction(for url: URL) {
    let controller = UIDocumentInteractionController(url: url)
    controller.delegate = self
    controller.presentOptionsMenu(from: view.bounds, in: view, animated: true)
}
```

**Короткий итог 2026**:
> `UIDocumentInteractionController` — системный контроллер для **просмотра**, **открытия в других приложениях**, **AirDrop**, **печати** и **поделиться** файлами.  
> В 2026 году:  
> - ключевые методы — `presentOpenInMenu`, `presentOptionsMenu`, `presentPreview`  
> - обязательно реализуйте `UIDocumentInteractionControllerDelegate` для предпросмотра  
> - идеален для PDF, DOCX, изображений, архивов и любых файлов  
> - в SwiftUI — заменяется на `ShareLink` / `FileRepresentation`  
> - это **надёжный** и **стандартный** способ интегрировать работу с файлами в UIKit  
