#messageui #email #mfmailcomposeviewcontroller #delegate #ios #result

---
## MFMailComposeResult

### Определение
**MFMailComposeResult** — это перечисление ([[enum]]) во фреймворке [[MessageUI]], которое определяет возможные результаты закрытия интерфейса составления электронного письма, представленного через [[MFMailComposeViewController]] . Оно сообщает делегату о том, какое действие предпринял пользователь при завершении работы с контроллером почты .

Этот тип данных используется исключительно в методе делегата `mailComposeController(_:didFinishWith:error:)`, который вызывается после того, как пользователь закончил взаимодействие с почтовым интерфейсом .

### Зачем это знать [[iOS]]-разработчику?
1.  **Обработка результата отправки:** Необходимо знать, было ли письмо отправлено, сохранено как черновик или пользователь отменил операцию.
2.  **Обратная связь пользователю:** Можно показывать уведомления (алерты) об успешной отправке или сохранении письма.
3.  **Аналитика:** Отслеживание, как часто пользователи отправляют письма или сохраняют их как черновики.
4.  **Обработка ошибок:** Реагирование на неудачные попытки отправки с анализом параметра `error`.

---

### Возможные значения

Перечисление `MFMailComposeResult` содержит четыре возможных значения, представленных как case'ы :

| Case | Значение (Raw Value) | Описание |
|------|----------------------|----------|
| `cancelled` | 0 | Пользователь отменил операцию (нажал "Отмена", затем "Удалить черновик" или просто закрыл контроллер) . |
| `saved` | 1 | Письмо было сохранено как черновик в папке "Черновики" почтового приложения . |
| `sent` | 2 | Письмо было поставлено в очередь на отправку (outbox) . **Важно:** Это не означает, что письмо было немедленно доставлено получателю. Это лишь означает, что оно успешно передано в системную очередь для последующей отправки . |
| `failed` | 3 | Произошла ошибка при отправке или сохранении письма. В этом случае параметр `error` метода делегата будет содержать информацию об ошибке . |

### Связь с делегатом и пример использования

`MFMailComposeResult` передается в качестве параметра `result` в метод делегата `MFMailComposeViewControllerDelegate` :

```swift
func mailComposeController(_ controller: MFMailComposeViewController, 
                           didFinishWith result: MFMailComposeResult, 
                           error: Error?) {
    // Обработка результата
}
```

#### Уровень 1: Базовая обработка с закрытием контроллера

Самый простой и распространенный сценарий — просто закрыть контроллер после завершения работы пользователя .

```swift
import UIKit
import MessageUI

class ViewController: UIViewController, MFMailComposeViewControllerDelegate {

    @IBAction func sendEmailButtonTapped(_ sender: UIButton) {
        guard MFMailComposeViewController.canSendMail() else {
            print("Устройство не может отправлять почту")
            return
        }

        let mailComposer = MFMailComposeViewController()
        mailComposer.mailComposeDelegate = self
        mailComposer.setToRecipients(["test@example.com"])
        mailComposer.setSubject("Тема письма")
        mailComposer.setMessageBody("Текст письма", isHTML: false)

        present(mailComposer, animated: true, completion: nil)
    }

    // MARK: - MFMailComposeViewControllerDelegate
    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {
        // Всегда закрываем контроллер
        controller.dismiss(animated: true, completion: nil)
    }
}
```

#### Уровень 2: Обработка с обратной связью для пользователя

Более продвинутый пример, где мы анализируем результат и показываем пользователю соответствующее уведомление.

```swift
import UIKit
import MessageUI

class FeedbackEmailViewController: UIViewController, MFMailComposeViewControllerDelegate {

    // ... остальной код ...

    func mailComposeController(_ controller: MFMailComposeViewController,
                               didFinishWith result: MFMailComposeResult,
                               error: Error?) {

        // Закрываем контроллер
        controller.dismiss(animated: true) { [weak self] in
            guard let self = self else { return }

            // Анализируем результат после закрытия
            switch result {
            case .cancelled:
                print("Отправка отменена пользователем")
                // Обычно не показываем уведомление на отмену

            case .saved:
                self.showAlert(title: "Черновик сохранен",
                               message: "Письмо сохранено в черновиках")

            case .sent:
                self.showAlert(title: "Письмо отправлено",
                               message: "Ваше письмо поставлено в очередь на отправку")

            case .failed:
                if let error = error {
                    self.showAlert(title: "Ошибка",
                                   message: "Не удалось отправить письмо: \(error.localizedDescription)")
                } else {
                    self.showAlert(title: "Ошибка",
                                   message: "Не удалось отправить письмо по неизвестной причине")
                }

            @unknown default:
                // На случай, если в будущем добавят новые case'ы
                print("Неизвестный результат: \(result)")
            }
        }
    }

    private func showAlert(title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

#### Уровень 3: Интеграция с SwiftUI

При использовании [[SwiftUI]] результат можно передавать обратно через `@Binding` или `Result` тип .

```swift
import SwiftUI
import MessageUI

struct MailComposerView: UIViewControllerRepresentable {
    @Binding var result: Result<MFMailComposeResult, Error>?
    @Environment(\.presentationMode) var presentationMode

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    func makeUIViewController(context: Context) -> MFMailComposeViewController {
        let mailComposer = MFMailComposeViewController()
        mailComposer.mailComposeDelegate = context.coordinator
        mailComposer.setToRecipients(["recipient@example.com"])
        mailComposer.setSubject("Тема")
        mailComposer.setMessageBody("Текст", isHTML: false)
        return mailComposer
    }

    func updateUIViewController(_ uiViewController: MFMailComposeViewController, context: Context) {}

    class Coordinator: NSObject, MFMailComposeViewControllerDelegate {
        var parent: MailComposerView

        init(_ parent: MailComposerView) {
            self.parent = parent
        }

        func mailComposeController(_ controller: MFMailComposeViewController,
                                   didFinishWith result: MFMailComposeResult,
                                   error: Error?) {
            parent.presentationMode.wrappedValue.dismiss()

            if let error = error {
                parent.result = .failure(error)
            } else {
                parent.result = .success(result)
            }
        }
    }
}

// Использование:
struct ContentView: View {
    @State private var isShowingMailView = false
    @State private var mailResult: Result<MFMailComposeResult, Error>?

    var body: some View {
        Button("Написать письмо") {
            isShowingMailView = true
        }
        .sheet(isPresented: $isShowingMailView) {
            MailComposerView(result: $mailResult)
        }
        .onChange(of: mailResult) { result in
            if let result = result {
                switch result {
                case .success(let composeResult):
                    switch composeResult {
                    case .sent:
                        print("Отправлено!")
                    case .saved:
                        print("Сохранено!")
                    case .cancelled:
                        print("Отменено")
                    case .failed:
                        print("Ошибка")
                    @unknown default:
                        break
                    }
                case .failure(let error):
                    print("Ошибка: \(error.localizedDescription)")
                }
            }
        }
    }
}
```

---

### Важные нюансы

1.  **`sent` ≠ доставлено:** Значение `.sent` гарантирует только то, что письмо было поставлено в очередь системой. Фактическая доставка зависит от многих факторов и не может быть отслежена через этот [[API]] .
2.  **Проверка возможности отправки:** Всегда вызывайте `MFMailComposeViewController.canSendMail()` перед попыткой представить контроллер. На устройствах без настроенной почты (симулятор, iPad без Mail.app) этот метод вернет `false`, и попытка создать контроллер приведет к крашу .
3.  **Обязательное закрытие контроллера:** В методе делегата необходимо вызвать `dismiss(animated:completion:)`, чтобы убрать почтовый интерфейс с экрана .
4.  **Обработка ошибок:** При получении `.failed` всегда проверяйте параметр `error` — он содержит детальную информацию о причине сбоя .
5.  **Исторические изменения:** В ранних версиях Swift (до iOS 9) `MFMailComposeResult` был структурой со свойством `value`. Начиная с iOS 9, он стал `enum`, соответствующим протоколу `RawRepresentable` .

### Итог
**MFMailComposeResult** — это простой, но важный элемент при работе с почтой в iOS. Он позволяет разработчику понять, какое действие предпринял пользователь после завершения работы с интерфейсом составления письма. Правильная обработка четырех возможных результатов (cancelled, saved, sent, failed) необходима для создания качественного пользовательского опыта и сбора аналитики.