**`MFMailComposeViewController`** — это класс из фреймворка **MessageUI**, который позволяет вашему приложению **открывать стандартный интерфейс составления письма** в почтовом клиенте [[iOS]] (Mail.app).

Это **единственный официальный способ** отправить email из приложения iOS с использованием **нативного интерфейса** Apple (с автозаполнением получателя, темы, тела, вложений и т.д.).

### Основные характеристики (актуально на 2026 год)

| Характеристика                     | Описание                                                                 | Важные замечания 2026 |
|------------------------------------|--------------------------------------------------------------------------|------------------------|
| **Фреймворк**                      | MessageUI                                                                | Требует импорта `import MessageUI` |
| **Класс**                          | `MFMailComposeViewController` : `UIViewController`                       | Наследуется от UIViewController |
| **Поддержка**                      | iOS 3.0+, macOS (через NSSharingService), но в iOS — основной способ     | Полностью поддерживается в iOS 18+ |
| **Требуется разрешение**           | Нет (но пользователь должен иметь настроенный почтовый аккаунт)          | Без аккаунта — покажет ошибку |
| **Делегирование**                  | Обязательно реализовать `MFMailComposeViewControllerDelegate`           | Для обработки результата отправки |
| **Альтернативы**                   | `UIApplication.open(URL)` с `mailto:` схемой                             | Не даёт нативный UI, хуже UX |

### Минимальный рабочий пример (самый частый паттерн 2026)

```swift
import UIKit
import MessageUI

@MainActor
class MailComposer: NSObject, MFMailComposeViewControllerDelegate {
    
    func presentMailComposer(from viewController: UIViewController) {
        guard MFMailComposeViewController.canSendMail() else {
            // Показать алерт: "Почта не настроена"
            showMailNotConfiguredAlert(on: viewController)
            return
        }
        
        let composer = MFMailComposeViewController()
        composer.mailComposeDelegate = self
        
        // Заполняем поля
        composer.setToRecipients(["support@example.com"])
        composer.setSubject("Обратная связь из приложения")
        composer.setMessageBody("Здравствуйте!\n\nОпишите вашу проблему или пожелание:", isHTML: false)
        
        // Вложения (опционально)
        if let imageData = UIImage(systemName: "star.fill")?.pngData() {
            composer.addAttachmentData(imageData, mimeType: "image/png", fileName: "screenshot.png")
        }
        
        viewController.present(composer, animated: true)
    }
    
    // Обязательный метод делегата
    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {
        controller.dismiss(animated: true)
        
        switch result {
        case .cancelled:
            print("Пользователь отменил отправку")
        case .saved:
            print("Письмо сохранено в черновиках")
        case .sent:
            print("Письмо успешно отправлено")
            // Можно показать благодарность
        case .failed:
            print("Ошибка отправки:", error?.localizedDescription ?? "Неизвестно")
        @unknown default:
            break
        }
    }
    
    private func showMailNotConfiguredAlert(on vc: UIViewController) {
        let alert = UIAlertController(
            title: "Почта не настроена",
            message: "Пожалуйста, настройте почтовый аккаунт в приложении Mail.",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        vc.present(alert, animated: true)
    }
}
```

### Как использовать в реальном приложении (рекомендуемый паттерн)

```swift
class SupportViewController: UIViewController {
    private let mailComposer = MailComposer()
    
    @IBAction func sendFeedbackTapped() {
        mailComposer.presentMailComposer(from: self)
    }
}
```

### Важные ограничения и поведение (2026)

- **Нет отправки без настроенного почтового аккаунта** — `canSendMail()` вернёт `false`
- **Пользователь может отменить/сохранить/отправить** — результат приходит в делегат
- **Вложения** — поддерживаются любые данные с правильным MIME-типом
- **HTML** — можно отправлять с `isHTML: true`
- **Фоновый режим** — не поддерживается (письмо отправляется только при активном приложении)
- **iOS 18+** — поведение идентично, но строже с Privacy Manifest (если используете другие функции [[Core Location]] / Contacts)
- **macOS** — лучше использовать `NSSharingService` с `NSSharingService.named(.composeEmail)`

### Лучшие практики MFMailComposeViewController в 2026

- **Всегда** проверяйте `MFMailComposeViewController.canSendMail()` перед показом
- **Обязательно** реализуйте делегат и обрабатывайте все 4 случая `MFMailComposeResult`
- **Держите** делегат как отдельный объект или extension контроллера — не делайте VC делегатом самого себя (retain cycle)
- **Используйте** `@MainActor` — все UI-операции должны быть на главном потоке
- **Для SwiftUI** — оборачивайте в `UIViewControllerRepresentable`
- **Альтернатива** — если не нужен нативный UI → используйте `mailto:` URL + `UIApplication.shared.open()`
- **Документируйте** — пишите комментарий «MFMailComposeViewController — отправка обратной связи через системный Mail»

**Короткий итог 2026**:
> `MFMailComposeViewController` — **единственный** способ показать **нативный интерфейс составления письма** в iOS.  
> В 2026 году:  
> - проверяйте `canSendMail()`  
> - реализуйте делегат и все 4 результата  
> - используйте как отдельный сервис-класс  
> - для SwiftUI — через `UIViewControllerRepresentable`  
> Это **стандартный и надёжный** способ отправки email из приложения.
