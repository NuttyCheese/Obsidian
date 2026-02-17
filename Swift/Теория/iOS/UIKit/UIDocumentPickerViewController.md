**UIDocumentPickerViewController** — это системный контроллер в [[UIKit]], который позволяет пользователю **выбирать файлы** из различных источников на устройстве или в облаке. Он открывает стандартный интерфейс [[iOS]] для работы с файлами (аналог «Открыть» в приложениях Apple).

Это **единственный рекомендуемый** способ в UIKit-приложениях дать пользователю выбрать документ, изображение, PDF, архив и т.д. в 2026 году.

### Основные возможности (актуально на iOS 19 / 2026)

| Возможность                              | Описание                                                                 | Значение по умолчанию / Рекомендация 2026 |
|------------------------------------------|--------------------------------------------------------------------------|--------------------------------------------|
| **documentTypes**                        | Массив UTType (какие файлы можно выбрать)                                | Обязательно указывать!                     |
| **allowsMultipleSelection**              | Разрешить выбор нескольких файлов                                        | `false` — чаще всего достаточно одного     |
| **shouldShowFileProviders**              | Показывать облачные сервисы (iCloud, Dropbox, Google Drive и т.д.)      | `true` — почти всегда                      |
| **directoryURL**                         | Начальная папка (локальная или iCloud)                                   | Редко используется                         |
| **popoverPresentationController**        | Настройка popover на iPad / Mac Catalyst                                 | Обязательно для iPad                       |
| **delegate**                             | `UIDocumentPickerDelegate` — уведомления о выборе или отмене             | Обязательно реализуйте                     |

### Самый популярный и современный паттерн 2026 года

#### 1. Выбор одного PDF-файла (самый частый случай)

```swift
class ImportViewController: UIViewController, UIDocumentPickerDelegate {
    
    @IBAction func importPDF(_ sender: UIButton) {
        // 1. Определяем разрешённые типы (только PDF)
        let pdfType = UTType.pdf.identifier
        
        let picker = UIDocumentPickerViewController(forOpeningContentTypes: [UTType.pdf])
        picker.delegate = self
        picker.allowsMultipleSelection = false
        picker.modalPresentationStyle = .formSheet  // красивее на iPad
        
        // Для iPad — обязательно указать sourceView
        picker.popoverPresentationController?.sourceView = sender
        picker.popoverPresentationController?.sourceRect = sender.bounds
        picker.popoverPresentationController?.permittedArrowDirections = .any
        
        present(picker, animated: true)
    }
    
    // Делегат: файл выбран
    func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
        guard let url = urls.first else { return }
        
        // Важно: для доступа к файлу нужно взять security-scoped bookmark
        guard url.startAccessingSecurityScopedResource() else { return }
        defer { url.stopAccessingSecurityScopedResource() }
        
        do {
            let data = try Data(contentsOf: url)
            print("PDF загружен, размер: \(data.count) байт")
            
            // Дальше: сохранить, обработать, показать в PDFView и т.д.
        } catch {
            print("Ошибка чтения файла: \(error)")
        }
    }
    
    // Делегат: пользователь отменил выбор
    func documentPickerWasCancelled(_ controller: UIDocumentPickerViewController) {
        print("Выбор файла отменён")
    }
}
```

#### 2. Выбор нескольких изображений (фото/скриншоты)

```swift
func pickMultipleImages() {
    let picker = UIDocumentPickerViewController(forOpeningContentTypes: [UTType.image])
    picker.delegate = self
    picker.allowsMultipleSelection = true  // ← ключевой флаг
    
    present(picker, animated: true)
}

func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
    for url in urls {
        guard url.startAccessingSecurityScopedResource() else { continue }
        defer { url.stopAccessingSecurityScopedResource() }
        
        if let image = UIImage(contentsOfFile: url.path) {
            // Добавляем в коллекцию, сохраняем и т.д.
            print("Выбрано изображение: \(image.size)")
        }
    }
}
```

#### 3. Полный список самых частых UTType (2026)

| Тип файла                     | UTType.identifier                              | Когда использовать |
|-------------------------------|------------------------------------------------|---------------------|
| PDF                           | `public.pdf`                                   | Документы, счета, контракты |
| Изображения (все)             | `public.image`                                 | Фото, скриншоты, мемы |
| JPEG / PNG / GIF              | `public.jpeg`, `public.png`, `com.compuserve.gif` | Если нужен конкретный формат |
| Текст (txt, md, csv и т.д.)   | `public.plain-text`, `public.text`             | Заметки, логи, экспорт |
| ZIP / архивы                  | `public.zip-archive`                           | Импорт данных |
| Все файлы                     | `public.data` или `public.content`             | Когда нужно всё подряд (редко) |

### Лучшие практики UIDocumentPickerViewController в Swift 2026

- **Всегда** указывай конкретные `UTType` — не используй `public.data`, иначе пользователь увидит все файлы  
- **security-scoped resource** — **обязательно** используй `startAccessingSecurityScopedResource()` / `stopAccessingSecurityScopedResource()` для доступа к файлу  
- **iPad / Mac Catalyst** — **обязательно** настрой `popoverPresentationController`, иначе краш  
- **allowsMultipleSelection** — включай только если действительно нужно несколько файлов  
- **Не презентуй на весь экран** — лучше `.formSheet` или popover  
- **[[@MainActor]]** — презентуй и обрабатывай результат на главном акторе  
- **Swift 6 strict concurrency** — полностью безопасен  
- **Документируйте** — пиши комментарий «UIDocumentPickerViewController — выбор PDF-файла с security scoping»

**Короткий девиз 2026**:
> UIDocumentPickerViewController — это **нативный системный файл-пикер** Apple: красивый, безопасный, поддерживает iCloud, Dropbox, Google Drive и все типы файлов.  
> В 2026 году это **единственный правильный** способ дать пользователю выбрать документ, фото, PDF или архив.  
> Всегда указывай конкретные UTType, обрабатывай security scoping и настраивай popover на iPad.
