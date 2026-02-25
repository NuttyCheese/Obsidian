**UITextViewDelegate** — это протокол в [[UIKit]], который позволяет **отслеживать и управлять** всеми основными событиями, происходящими в **[[UITextView]]**.

Он используется для:
- реакции на начало/окончание редактирования
- контроля ввода текста (ограничение символов, фильтрация)
- реализации placeholder
- отслеживания изменений текста и выделения
- обработки нажатий на ссылки и другие интерактивные элементы

В 2026 году это **один из самых часто используемых** делегатов в UIKit-приложениях, особенно в сценариях заметок, чатов, комментариев, форм и редакторов текста.

### Основные методы UITextViewDelegate (актуальные в 2026)

| Метод делегата                                      | Когда вызывается                                                                 | Что обычно делают внутри метода                                      | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------|
| `textViewDidBeginEditing(_:)`                       | После того, как текстовое поле стало первым респондером (появилась клавиатура) | Показ placeholder → скрыть, анимация, логирование                    | Скрытие placeholder |
| `textViewDidEndEditing(_:)`                         | После того, как текстовое поле потеряло фокус (клавиатура скрыта)              | Сохранение текста, валидация, скрытие клавиатуры                      | Сохранение черновика |
| `textViewDidChange(_:)`                             | При каждом изменении текста (вставка, удаление символа)                         | Обновление счётчика символов, валидация в реальном времени, поиск     | Счётчик символов, live-поиск |
| `textViewDidChangeSelection(_:)`                    | При изменении позиции курсора или выделения текста                              | Реакция на выделение (форматирование, контекстное меню)               | Кастомные toolbar при выделении |
| `textView(_:shouldChangeTextIn:replacementText:)`   | Перед любым изменением текста (вставка, удаление, paste)                        | Ограничение длины, фильтрация символов, обработка enter               | Ограничение 280 символов (как в Twitter) |
| `textView(_:shouldInteractWith:in:interaction:)`    | При нажатии на ссылку, текст с data-detector или attachment                     | Разрешение/запрет открытия URL, телефона, адреса                     | Открытие ссылок в SFSafariViewController |
| `textView(_:shouldInteractWithTextAttachment:in:interaction:)` | При взаимодействии с вложениями (attachment)                             | Кастомная обработка вложений (изображения, файлы)                     | Редко, в редакторах |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[UITextView]] + UITextViewDelegate + [[Combine]] + динамическая высота + placeholder)

```swift
import UIKit
import Combine

class CommentInputViewController: UIViewController, UITextViewDelegate {
    
    private let viewModel = CommentViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var textView: UITextView = {
        let tv = UITextView()
        tv.font = .systemFont(ofSize: 17)
        tv.textColor = .label
        tv.isScrollEnabled = false                // Для динамической высоты
        tv.textContainerInset = UIEdgeInsets(top: 12, left: 8, bottom: 12, right: 8)
        tv.layer.borderColor = UIColor.systemGray4.cgColor
        tv.layer.borderWidth = 1
        tv.layer.cornerRadius = 12
        tv.delegate = self
        tv.translatesAutoresizingMaskIntoConstraints = false
        return tv
    }()
    
    private lazy var placeholderLabel: UILabel = {
        let lbl = UILabel()
        lbl.text = "Напишите комментарий..."
        lbl.font = .systemFont(ofSize: 17)
        lbl.textColor = .systemGray
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    private lazy var sendButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Отправить", for: .normal)
        btn.titleLabel?.font = .systemFont(ofSize: 17, weight: .medium)
        btn.isEnabled = false
        btn.translatesAutoresizingMaskIntoConstraints = false
        return btn
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        view.addSubview(textView)
        textView.addSubview(placeholderLabel)
        view.addSubview(sendButton)
        
        NSLayoutConstraint.activate([
            textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
            textView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            textView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            textView.heightAnchor.constraint(greaterThanOrEqualToConstant: 50),
            
            placeholderLabel.topAnchor.constraint(equalTo: textView.topAnchor, constant: 12),
            placeholderLabel.leadingAnchor.constraint(equalTo: textView.leadingAnchor, constant: 12),
            
            sendButton.topAnchor.constraint(equalTo: textView.bottomAnchor, constant: 12),
            sendButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Текст → ViewModel
        textView.publisher(for: \.text)
            .removeDuplicates()
            .assign(to: \.commentText, on: viewModel)
            .store(in: &cancellables)
        
        // ViewModel → UI
        viewModel.$commentText
            .map { $0.isEmpty }
            .assign(to: \.isHidden, on: placeholderLabel)
            .store(in: &cancellables)
        
        viewModel.$commentText
            .map { !$0.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty }
            .assign(to: \.isEnabled, on: sendButton)
            .store(in: &cancellables)
    }
    
    // MARK: - UITextViewDelegate
    
    func textViewDidBeginEditing(_ textView: UITextView) {
        placeholderLabel.isHidden = true
    }
    
    func textViewDidEndEditing(_ textView: UITextView) {
        placeholderLabel.isHidden = textView.text.isEmpty
        // Можно сохранить черновик
    }
    
    func textViewDidChange(_ textView: UITextView) {
        // Динамическая высота
        textView.invalidateIntrinsicContentSize()
    }
    
    func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
        let maxLength = 500
        let currentText = textView.text ?? ""
        guard let stringRange = Range(range, in: currentText) else { return false }
        let updatedText = currentText.replacingCharacters(in: stringRange, with: text)
        return updatedText.count <= maxLength
    }
    
    func textView(_ textView: UITextView, shouldInteractWith URL: URL, in characterRange: NSRange, interaction: UITextItemInteraction) -> Bool {
        // Открываем ссылки в SFSafariViewController
        UIApplication.shared.open(URL)
        return false
    }
}

// ViewModel
@MainActor
class CommentViewModel: ObservableObject {
    @Published var commentText: String = ""
}
```

### Лучшие практики UITextViewDelegate в 2026 году

- **Слабая ссылка** — делайте `weak var delegate: UITextViewDelegate?` в кастомных UITextView  
- **Динамическая высота** — отключайте `isScrollEnabled = false` и используйте `invalidateIntrinsicContentSize()`  
- **Placeholder** — добавляйте отдельный `UILabel` и скрывайте при вводе текста  
- **Ограничение длины** — реализуйте в `shouldChangeTextIn` — это самый надёжный способ  
- **Для Combine** — используйте `publisher(for: \.text)` или `publisher(for: \.attributedText)`  
- **Для SwiftUI** — используйте `TextEditor` — UITextViewDelegate нужен только в UIKit  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint`  
- **Документируйте** — пишите комментарий:

```swift
extension CommentInputViewController: UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        // Обновляем высоту и UI
        textView.invalidateIntrinsicContentSize()
    }
}
```

**Короткий итог 2026**:
> `UITextViewDelegate` — протокол для **обработки всех событий** UITextView (ввод, выделение, ссылки).  
> В 2026 году:  
> - ключевые методы — `textViewDidChange`, `shouldChangeTextIn`, `textViewDidBeginEditing/EndEditing`  
> - самый популярный паттерн — динамическая высота + placeholder + ограничение символов + Combine  
> - идеален для комментариев, заметок, чатов, форм с длинным текстом  
> - в SwiftUI — эквивалент `TextEditor` с `.onChange`  
> - это **фундаментальный** делегат для любого приложения с текстовым вводом  
