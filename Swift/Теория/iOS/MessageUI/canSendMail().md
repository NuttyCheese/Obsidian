**`MFMailComposeViewController.canSendMail()`** — это **статический метод** класса `MFMailComposeViewController`, который проверяет, **может ли устройство отправить email** через системное приложение Почта (Mail.app).

### Что именно возвращает `canSendMail()`

| Возвращаемое значение | Когда возвращает `true`                                      | Когда возвращает `false`                                      |
|-----------------------|---------------------------------------------------------------|---------------------------------------------------------------|
| `true`                | На устройстве настроен хотя бы один почтовый аккаунт (iCloud, Gmail, Яндекс, корпоративный и т.д.) | Нет ни одного настроенного почтового аккаунта в приложении «Почта» |
| `false`               | —                                                             | Устройство без настроенной почты (новый iPhone, детский режим, enterprise-ограничения и т.д.) |

### Самые важные моменты 2026 года

- Метод **не проверяет интернет** — только наличие настроенного аккаунта  
- Метод **не запрашивает разрешения** — работает мгновенно, без алертов  
- Вызывать его нужно **перед** созданием `MFMailComposeViewController`  
- Если `canSendMail() == false` → показывайте альтернативу (например, кнопку «Написать нам на support@yourapp.com»)

### Рекомендуемый паттерн использования (2026 стандарт)

```swift
import MessageUI

@MainActor
func showMailComposer(from vc: UIViewController) {
    guard MFMailComposeViewController.canSendMail() else {
        // Показываем альтернативу
        showNoMailAccountAlert(on: vc)
        return
    }
    
    let composer = MFMailComposeViewController()
    composer.mailComposeDelegate = self  // или отдельный делегат
    
    composer.setToRecipients(["support@yourapp.com"])
    composer.setSubject("Обратная связь из приложения")
    composer.setMessageBody("Версия: \(Bundle.main.appVersion)\n\nВаш отзыв:", isHTML: false)
    
    vc.present(composer, animated: true)
}

private func showNoMailAccountAlert(on vc: UIViewController) {
    let alert = UIAlertController(
        title: "Почта не настроена",
        message: "На вашем устройстве нет настроенного почтового аккаунта. Пожалуйста, настройте почту в приложении «Почта», чтобы отправить сообщение.",
        preferredStyle: .alert
    )
    
    alert.addAction(UIAlertAction(title: "Настройки", style: .default) { _ in
        if let url = URL(string: UIApplication.openSettingsURLString) {
            UIApplication.shared.open(url)
        }
    })
    
    alert.addAction(UIAlertAction(title: "Написать вручную", style: .default) { _ in
        if let url = URL(string: "mailto:support@yourapp.com?subject=Обратная связь") {
            UIApplication.shared.open(url)
        }
    })
    
    alert.addAction(UIAlertAction(title: "Отмена", style: .cancel))
    
    vc.present(alert, animated: true)
}
```

### Полный пример с делегатом

```swift
class FeedbackViewController: UIViewController, MFMailComposeViewControllerDelegate {
    
    @IBAction func sendFeedbackButtonTapped() {
        guard MFMailComposeViewController.canSendMail() else {
            showNoMailAccountAlert()
            return
        }
        
        let composer = MFMailComposeViewController()
        composer.mailComposeDelegate = self
        
        composer.setToRecipients(["feedback@yourapp.com"])
        composer.setSubject("Обратная связь — версия \(Bundle.main.appVersion)")
        composer.setMessageBody("Опишите проблему или пожелание:\n\n", isHTML: false)
        
        present(composer, animated: true)
    }
    
    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {
        controller.dismiss(animated: true)
        
        switch result {
        case .sent:
            showSuccessAlert()
        case .failed:
            showErrorAlert(error)
        case .cancelled, .saved:
            break
        @unknown default:
            break
        }
    }
    
    private func showSuccessAlert() {
        let alert = UIAlertController(title: "Отправлено!", message: "Спасибо за ваш отзыв.", preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    private func showErrorAlert(_ error: Error?) {
        let message = error?.localizedDescription ?? "Не удалось отправить письмо."
        let alert = UIAlertController(title: "Ошибка", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

### Лучшие практики `canSendMail()` в 2026 году

- **Всегда** проверяйте `canSendMail()` **перед** созданием контроллера — иначе получите пустой или неработающий пикер  
- **Показывайте альтернативу**, если `false` — mailto-ссылка или копирование email в буфер  
- **Не полагайтесь** на `canSendMail()` как на проверку интернета — оно не связано с сетью  
- **Для SwiftUI** — оборачивайте `MFMailComposeViewController` в `UIViewControllerRepresentable` и проверяйте `canSendMail()` перед `.sheet`  
- **Privacy Manifest** — **не требуется** для MessageUI (это UI-компонент, не использует приватные данные)  
- **Документируйте** — пишите комментарий «MFMailComposeViewController.canSendMail() — проверка наличия настроенного почтового аккаунта перед показом composer»

**Короткий итог 2026**:
> `MFMailComposeViewController.canSendMail()` — это **быстрая проверка**, может ли устройство отправить email через системную Почту.  
> В 2026 году:  
> - возвращает `true`, если настроен хотя бы один почтовый аккаунт  
> - **не проверяет интернет** и **не запрашивает разрешения**  
> - **обязательно** вызывайте перед созданием `MFMailComposeViewController`  
> - если `false` — показывайте альтернативу (mailto или копирование email)  
> Это **единственный надёжный** способ избежать пустого/сломанного пикера отправки почты.

Удачи с надёжной и вежливой обратной связью в твоём приложении! ✉️