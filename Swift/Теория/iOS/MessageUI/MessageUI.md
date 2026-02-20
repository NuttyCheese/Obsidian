**MessageUI** — это фреймворк в [[iOS]] / iPadOS, который позволяет приложению открывать **стандартный интерфейс составления письма** через системное приложение Почта (Mail.app).

С 2010-х годов и по 2026 год это **единственный официальный и рекомендуемый** способ отправить email из приложения с использованием **нативного UI Apple** (поля «Кому», «Тема», тело, вложения, подпись пользователя и т.д.).

### Ключевые компоненты фреймворка MessageUI (актуально на 2026 год)

| Компонент                               | Что это                                                         | Самый частый сценарий использования      |
| --------------------------------------- | --------------------------------------------------------------- | ---------------------------------------- |
| [[MFMailComposeViewController]]         | Контроллер для показа окна составления письма                   | Основной инструмент отправки email       |
| [[MFMailComposeViewControllerDelegate]] | Протокол-делегат для обработки результата отправки              | Обязателен при использовании контроллера |
| [[MFMailComposeResult]]                 | Enum с результатами (cancelled, saved, sent, failed)            | Обработка в делегате                     |
| [[canSendMail()]]                       | Статический метод — проверяет, настроена ли почта на устройстве | Обязательная проверка перед показом      |

### Минимальный и рекомендуемый шаблон 2026 года (UIKit)

```swift
import UIKit
import MessageUI

@MainActor
final class MailComposer: NSObject, MFMailComposeViewControllerDelegate {
    
    weak var presentingVC: UIViewController?
    
    func presentMail(from vc: UIViewController,
                     to: [String] = ["support@yourapp.com"],
                     subject: String = "Обратная связь",
                     body: String = "Опишите проблему или пожелание:",
                     isHTML: Bool = false) {
        
        guard MFMailComposeViewController.canSendMail() else {
            showCannotSendAlert(on: vc)
            return
        }
        
        self.presentingVC = vc
        
        let composer = MFMailComposeViewController()
        composer.mailComposeDelegate = self
        
        composer.setToRecipients(to)
        composer.setSubject(subject)
        composer.setMessageBody(body, isHTML: isHTML)
        
        // Пример вложения
        if let data = UIImage(systemName: "star.fill")?.pngData() {
            composer.addAttachmentData(data, mimeType: "image/png", fileName: "screenshot.png")
        }
        
        vc.present(composer, animated: true)
    }
    
    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {
        
        controller.dismiss(animated: true)
        
        switch result {
        case .cancelled:
            print("Отправка отменена")
            
        case .saved:
            print("Сохранено в черновики")
            
        case .sent:
            // Самый приятный UX
            let alert = UIAlertController(
                title: "Отправлено!",
                message: "Спасибо за ваш отзыв. Мы свяжемся с вами.",
                preferredStyle: .alert
            )
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            presentingVC?.present(alert, animated: true)
            
        case .failed:
            let message = error?.localizedDescription ?? "Не удалось отправить. Проверьте интернет и настройки почты."
            let alert = UIAlertController(
                title: "Ошибка",
                message: message,
                preferredStyle: .alert
            )
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            presentingVC?.present(alert, animated: true)
            
        @unknown default:
            break
        }
        
        presentingVC = nil
    }
    
    private func showCannotSendAlert(on vc: UIViewController) {
        let alert = UIAlertController(
            title: "Почта не настроена",
            message: "Настройте почтовый аккаунт в приложении «Почта».",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        vc.present(alert, animated: true)
    }
}
```

### Как использовать в контроллере

```swift
class FeedbackViewController: UIViewController {
    private let composer = MailComposer()
    
    @IBAction func sendFeedbackTapped() {
        composer.presentMail(from: self,
                             subject: "Обратная связь из приложения",
                             body: "Версия: \(Bundle.main.appVersion)\n\nВаш отзыв:")
    }
}
```

### Лучшие практики MessageUI в 2026 году

- **Всегда** проверяйте `MFMailComposeViewController.canSendMail()` перед созданием контроллера  
- **Обязательно** реализуйте делегат и обработайте **все 4** значения `MFMailComposeResult`  
- **Держите делегат** отдельно (не делайте [[UIViewController]] делегатом самого себя — [[retain cycle]])  
- **В `.sent`** — показывайте благодарность (алерт, тост, галочку)  
- **В `.failed`** — информируйте и давайте возможность повторить  
- **Для SwiftUI** — используйте [[UIViewControllerRepresentable]] + [[Coordinator]]  
- **Нет необходимости** в Info.plist ключах (кроме случая, если используете другие функции)  
- **Документируйте** — пишите комментарий «MFMailComposeViewController — отправка обратной связи через системное приложение Почта»

**Короткий итог 2026**:
> **MessageUI** — фреймворк для показа **нативного окна составления письма** в iOS.  
> В 2026 году:  
> - основной класс — `MFMailComposeViewController`  
> - обязательный делегат — `MFMailComposeViewControllerDelegate`  
> - ключевой метод — `mailComposeController(_:didFinishWith:error:)`  
> - проверка — `canSendMail()`  
> Это **единственный официальный** и **самый ожидаемый пользователем** способ отправить email из приложения.
