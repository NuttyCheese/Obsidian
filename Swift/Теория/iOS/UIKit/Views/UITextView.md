**UITextView** — это многострочное текстовое поле в [[UIKit]], предназначенное для ввода, редактирования и отображения текста большого объёма.  
В 2026 году это всё ещё основной инструмент для реализации полей заметок, чатов, комментариев, описаний продуктов, юридических текстов и любых сценариев, где нужен многострочный ввод.

### Когда использовать UITextView в 2026 году (реальные кейсы)

| Сценарий                            | Почему именно UITextView (а не [[UITextField]])   | Альтернатива в SwiftUI           |
| ----------------------------------- | ------------------------------------------------- | -------------------------------- |
| Заметки, дневник, черновик письма   | Многострочный ввод + прокрутка + атрибуты         | `TextEditor`                     |
| Поле комментария / отзыва           | Длинный текст + placeholder + автокоррект         | `TextEditor`                     |
| Чат (ввод сообщения)                | Многострочный + динамическая высота + placeholder | `TextEditor` + `.lineLimit(nil)` |
| Отображение форматированного текста | Поддержка `attributedText` (цвет, жирный, ссылки) | `Text` + `AttributedString`      |
| Редактирование Markdown / HTML      | Полная поддержка атрибутов + выделение            | `TextEditor` + markdown          |
| Юридический текст / договор         | Очень длинный текст + readonly-режим              | `Text` + `.scrollable()`         |

### Сравнение [[UITextView]] vs [[UITextField]] vs [[TextEditor]] (2026)

| Характеристика                      | UITextView ([[UIKit]])      | UITextField (UIKit)     | TextEditor ([[SwiftUI]])      |
| ----------------------------------- | --------------------------- | ----------------------- | ----------------------------- |
| Количество строк                    | Многострочный               | Однострочный            | Многострочный                 |
| Placeholder встроенный              | Нет (нужен костыль)         | Да                      | Нет (нужен костыль)           |
| AttributedText                      | Да (полная поддержка)       | Ограниченная            | Да (через AttributedString)   |
| Автоматическая высота               | Нужно вручную (constraints) | Не нужно                | Автоматическая                |
| Прокрутка                           | Встроенная                  | Нет                     | Встроенная                    |
| Делегат / события                   | [[UITextViewDelegate]]      | [[UITextFieldDelegate]] | `.onChange`, `.focused`       |
| Производительность на 10 000+ строк | Отличная                    | —                       | Хуже на очень больших текстах |
| Минимальная версия                  | iOS 3+                      | iOS 2+                  | iOS 14+                       |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UITextView + [[Auto Layout]] + [[Combine]] + динамическая высота)

```swift
import UIKit
import Combine

class CommentInputViewController: UIViewController {
    
    private let viewModel = CommentViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var textView: UITextView = {
        let tv = UITextView()
        tv.font = .systemFont(ofSize: 17)
        tv.textColor = .label
        tv.isScrollEnabled = false          // Важно для динамической высоты
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
            textView.heightAnchor.constraint(greaterThanOrEqualToConstant: 50), // минимальная высота
            
            placeholderLabel.topAnchor.constraint(equalTo: textView.topAnchor, constant: 12),
            placeholderLabel.leadingAnchor.constraint(equalTo: textView.leadingAnchor, constant: 12),
            
            sendButton.topAnchor.constraint(equalTo: textView.bottomAnchor, constant: 12),
            sendButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Двусторонняя связь text ↔ ViewModel
        textView.publisher(for: \.text)
            .removeDuplicates()
            .assign(to: \.commentText, on: viewModel)
            .store(in: &cancellables)
        
        viewModel.$commentText
            .map { $0.isEmpty }
            .assign(to: \.isHidden, on: placeholderLabel)
            .store(in: &cancellables)
        
        viewModel.$commentText
            .map { !$0.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty }
            .assign(to: \.isEnabled, on: sendButton)
            .store(in: &cancellables)
        
        // Динамическая высота textView
        textView.publisher(for: \.contentSize)
            .map { $0.height + 24 } // + insets
            .assign(to: \.constant, on: textView.constraints.first { $0.firstAttribute == .height }!)
            .store(in: &cancellables)
    }
}

extension CommentInputViewController: UITextViewDelegate {
    func textViewDidChange(_ textView: UITextView) {
        // Можно дополнительно обновлять layout
        textView.invalidateIntrinsicContentSize()
    }
}

// ViewModel
@MainActor
class CommentViewModel: ObservableObject {
    @Published var commentText: String = ""
}
```

### Лучшие практики UITextView в 2026 году

- **Динамическая высота** — отключайте `isScrollEnabled = false` и используйте Auto Layout + `invalidateIntrinsicContentSize()`  
- **Placeholder** — добавляйте отдельный `UILabel` поверх и скрывайте при вводе текста  
- **Для атрибутов** — используйте `attributedText` для форматирования (жирный, курсив, ссылки)  
- **Для Combine** — подписывайтесь на `publisher(for: \.text)` или используйте `@Published` в ViewModel  
- **Для производительности** — на очень длинных текстах (10 000+ строк) используйте `textStorage` и `layoutManager` вручную  
- **Для SwiftUI** — используйте `TextEditor` — UITextView нужен только в UIKit или смешанных проектах  
- **Для доступности** — задавайте `accessibilityLabel` и `accessibilityHint`  
- **Документируйте** — пишите комментарий:

```swift
/// UITextView с динамической высотой и placeholder для ввода комментария
private lazy var commentTextView: UITextView = {
    let tv = UITextView()
    tv.font = .systemFont(ofSize: 17)
    tv.isScrollEnabled = false
    tv.textContainerInset = UIEdgeInsets(top: 12, left: 8, bottom: 12, right: 8)
    return tv
}()
```

**Короткий итог 2026**:
> `UITextView` — **многострочное поле ввода и отображения текста** в UIKit.  
> В 2026 году:  
> - ключевые возможности — `text`, `attributedText`, `isEditable`, `isSelectable`, делегат  
> - самый популярный паттерн — динамическая высота + placeholder + Combine  
> - идеален для заметок, комментариев, чатов, описаний  
> - в SwiftUI — заменяется на `TextEditor`  
> - это **фундаментальный** компонент для любого приложения с текстовым вводом  
