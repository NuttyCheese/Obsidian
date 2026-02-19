**`MFMailComposeViewControllerDelegate`** — это **протокол** из фреймворка **MessageUI**, который определяет методы-делегаты для обработки результатов работы **[[MFMailComposeViewController]]**.

Это **обязательный** протокол, если вы используете контроллер составления письма: без реализации делегата вы не сможете корректно обработать, что сделал пользователь (отправил письмо, отменил, сохранил черновик и т.д.).

### Основные методы протокола (актуально на 2026 год)

| Метод делегата                                                                 | Когда вызывается                                                                 | Что нужно делать внутри метода                                                                 |
|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| `mailComposeController(_:didFinishWith:error:)`                                | Пользователь завершил работу с контроллером (нажал Отправить / Отмена / Сохранить) | Обязательно вызвать `dismiss(animated:)` и обработать результат `MFMailComposeResult`           |
| `mailComposeControllerDidFinishWithResult(_:)` (устаревший, до iOS 14)         | —                                                                                | Использовать только в legacy-коде                                                               |

### Единственный обязательный метод (современный стандарт)

```swift
func mailComposeController(_ controller: MFMailComposeViewController,
                           didFinishWith result: MFMailComposeResult,
                           error: Error?)
```

**Возможные значения `result` (enum `MFMailComposeResult`)**

| Значение           | Что означает                                                 | Что обычно делать в приложении                          |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------- |
| `.cancelled`       | Пользователь отменил составление письма                      | Просто закрыть контроллер, ничего не показывать         |
| `.saved`           | Письмо сохранено в черновики                                 | Можно показать «Сохранено в черновиках» (редко нужно)   |
| `.sent`            | Письмо успешно отправлено                                    | Показать благодарность («Спасибо за обратную связь!»)   |
| `.failed`          | Ошибка отправки (нет интернета, проблема с аккаунтом и т.д.) | Показать алерт с ошибкой и предложить попробовать снова |
| `@unknown default` | Будущие случаи (новые версии [[iOS]])                        | Просто закрыть контроллер                               |

### Полный рекомендуемый шаблон реализации делегата (2026)

```swift
import UIKit
import MessageUI

@MainActor
final class MailHelper: NSObject, MFMailComposeViewControllerDelegate {
    
    weak var presentingViewController: UIViewController?
    
    func presentMail(from vc: UIViewController) {
        guard MFMailComposeViewController.canSendMail() else {
            showCannotSendMailAlert(on: vc)
            return
        }
        
        self.presentingViewController = vc
        
        let composer = MFMailComposeViewController()
        composer.mailComposeDelegate = self
        
        composer.setToRecipients(["support@yourapp.com"])
        composer.setSubject("Обратная связь / Ошибка в приложении")
        composer.setMessageBody("Версия приложения: \(Bundle.main.appVersion)\n\nОпишите проблему:", isHTML: false)
        
        vc.present(composer, animated: true)
    }
    
    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {
        controller.dismiss(animated: true)
        
        switch result {
        case .cancelled:
            // Ничего не делаем или показываем "Отменено"
            break
            
        case .saved:
            // Редко нужно обрабатывать
            break
            
        case .sent:
            showSuccessAlert(on: presentingViewController)
            
        case .failed:
            showFailureAlert(on: presentingViewController, error: error)
            
        @unknown default:
            break
        }
        
        // Очищаем weak-ссылку
        presentingViewController = nil
    }
    
    private func showSuccessAlert(on vc: UIViewController?) {
        let alert = UIAlertController(
            title: "Отправлено",
            message: "Спасибо за ваш отзыв!",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        vc?.present(alert, animated: true)
    }
    
    private func showFailureAlert(on vc: UIViewController?, error: Error?) {
        let message = error?.localizedDescription ?? "Не удалось отправить письмо"
        let alert = UIAlertController(
            title: "Ошибка",
            message: message,
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        vc?.present(alert, animated: true)
    }
    
    private func showCannotSendMailAlert(on vc: UIViewController) {
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

### Как правильно интегрировать в приложение (рекомендуемый подход)

```swift
class FeedbackViewController: UIViewController {
    private let mailHelper = MailHelper()
    
    @IBAction func sendFeedbackButtonTapped() {
        mailHelper.presentMail(from: self)
    }
}
```

### Важные замечания и лучшие практики 2026 года

- **Всегда** проверяйте `MFMailComposeViewController.canSendMail()` перед созданием контроллера  
- **Обязательно** реализуйте делегат и обрабатывайте **все** 4 случая `MFMailComposeResult`  
- **Держите делегат** как отдельный объект (не делайте [[UIViewController]] делегатом самого себя — [[retain cycle]])  
- **Используйте** [[@MainActor]] — все UI-операции и делегаты должны быть на главном потоке  
- **Для SwiftUI** — оборачивайте в `UIViewControllerRepresentable` + `Coordinator`  
- **Альтернатива** — если не нужен нативный интерфейс → используйте `mailto:` URL  
- **Privacy** — не требует специальных ключей в Info.plist (кроме случая, если вы используете другие функции)  
- **Документируйте** — пишите комментарий «MFMailComposeViewControllerDelegate — обработка результата отправки письма»

**Короткий итог 2026**:
> `MFMailComposeViewControllerDelegate` — **единственный** способ узнать, что сделал пользователь с окном составления письма.  
> В 2026 году:  
> - главный метод — `mailComposeController(_:didFinishWith:error:)`  
> - всегда обрабатывайте все 4 результата (.sent, .cancelled, .saved, .failed)  
> - используйте как отдельный helper-класс с weak-ссылкой на presenting VC  
> - проверяйте `canSendMail()` перед показом  
> Это **надёжный и ожидаемый** пользователем способ отправки email из приложения.
