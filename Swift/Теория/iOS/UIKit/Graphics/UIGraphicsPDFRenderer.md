**UIGraphicsPDFRenderer** — это класс в [[UIKit]] (с iOS 10, 2016 год), предназначенный для **программного создания PDF-документов** в [[iOS]]-приложениях.

Это современный и рекомендуемый способ генерации PDF (замена устаревшему `UIGraphicsBeginPDFContextToData` / `UIGraphicsEndPDFContext`).

Он работает по тому же принципу, что и **UIGraphicsImageRenderer**, но вместо растрового изображения создаёт **векторный PDF**.

### Основные преимущества UIGraphicsPDFRenderer (2026 актуально)

| Преимущество                          | Почему важно в 2026 году                                  |
| ------------------------------------- | --------------------------------------------------------- |
| Полностью векторный вывод             | PDF остаётся чётким при любом масштабе (масштабируемость) |
| Поддержка multi-page                  | Легко создавать документы из нескольких страниц           |
| Thread-safe                           | Безопасно генерировать PDF в фоне                         |
| Корректная работа с sRGB / Display P3 | Цвета не искажаются на современных экранах                |
| Простой API с замыканием              | Меньше ошибок, чем в старом CGPDFContext                  |
| Лёгкая интеграция с UIKit-рендерингом | Можно рендерить [[UIView]].layer прямо в PDF              |

### Основные способы использования UIGraphicsPDFRenderer

#### 1. Самый частый паттерн — создание многостраничного PDF из массива вью / контента

```swift
func generatePDF(from pages: [UIView]) -> Data? {
    let pageSize = CGSize(width: 595.28, height: 841.89) // A4 в points (72 dpi)
    
    let renderer = UIGraphicsPDFRenderer(bounds: CGRect(origin: .zero, size: pageSize))
    
    let data = renderer.pdfData { context in
        for pageView in pages {
            // Начинаем новую страницу
            context.beginPage()
            
            // Масштабируем view под размер страницы (если нужно)
            let scale = min(pageSize.width / pageView.bounds.width,
                           pageSize.height / pageView.bounds.height)
            
            let transform = CGAffineTransform(scaleX: scale, y: scale)
            context.cgContext.concatenate(transform)
            
            // Рендерим слой view в PDF-контекст
            pageView.layer.render(in: context.cgContext)
        }
    }
    
    return data
}
```

#### 2. PDF из текста, таблиц, изображений (самый гибкий способ)

```swift
func createInvoicePDF(customerName: String, items: [(name: String, price: Double)]) -> Data? {
    let pageSize = CGSize(width: 595, height: 842) // A4
    let margin: CGFloat = 40
    
    let renderer = UIGraphicsPDFRenderer(bounds: CGRect(origin: .zero, size: pageSize))
    
    let data = renderer.pdfData { context in
        context.beginPage()
        let cgContext = context.cgContext
        
        // Заголовок
        let title = "Счёт-фактура"
        let titleAttrs: [NSAttributedString.Key: Any] = [
            .font: UIFont.boldSystemFont(ofSize: 24),
            .foregroundColor: UIColor.black
        ]
        title.draw(at: CGPoint(x: margin, y: margin), withAttributes: titleAttrs)
        
        // Информация о клиенте
        let customerText = "Клиент: \(customerName)"
        customerText.draw(at: CGPoint(x: margin, y: margin + 40), withAttributes: [
            .font: UIFont.systemFont(ofSize: 16)
        ])
        
        // Таблица товаров
        var yOffset: CGFloat = margin + 80
        for item in items {
            let line = "\(item.name) — \(String(format: "%.2f ₽", item.price))"
            line.draw(at: CGPoint(x: margin, y: yOffset), withAttributes: [
                .font: UIFont.systemFont(ofSize: 14)
            ])
            yOffset += 24
        }
        
        // Итог
        let total = items.reduce(0) { $0 + $1.price }
        let totalText = "Итого: \(String(format: "%.2f ₽", total))"
        totalText.draw(at: CGPoint(x: margin, y: yOffset + 20), withAttributes: [
            .font: UIFont.boldSystemFont(ofSize: 16)
        ])
    }
    
    return data
}
```

#### 3. PDF из текущего экрана (скриншот в PDF)

```swift
func currentScreenToPDF() -> Data? {
    guard let window = UIApplication.shared.windows.first,
          let view = window.rootViewController?.view else { return nil }
    
    let pageSize = view.bounds.size
    let renderer = UIGraphicsPDFRenderer(bounds: view.bounds)
    
    let data = renderer.pdfData { context in
        context.beginPage()
        view.drawHierarchy(in: view.bounds, afterScreenUpdates: true)
    }
    
    return data
}
```

#### 4. Сохранение и открытие PDF

```swift
func saveAndSharePDF(data: Data) {
    let tempURL = FileManager.default.temporaryDirectory.appendingPathComponent("document.pdf")
    
    do {
        try data.write(to: tempURL)
        
        let activityVC = UIActivityViewController(activityItems: [tempURL], applicationActivities: nil)
        present(activityVC, animated: true)
    } catch {
        print("Ошибка сохранения PDF:", error)
    }
}
```

### Лучшие практики UIGraphicsPDFRenderer в Swift 2026

- **Всегда** используйте `UIGraphicsPDFRenderer` вместо старого `UIGraphicsBeginPDFContextToData`  
- **Задавайте** правильный размер страницы в **points** (A4 = 595 × 842 pt при 72 dpi)  
- **Для многостраничных документов** — вызывайте `context.beginPage()` перед каждой страницей  
- **Для рендеринга UIView** — используйте `view.layer.render(in:)` или `view.drawHierarchy`  
- **Для текста** — используйте [[NSAttributedString]] + `draw(at:withAttributes:)`  
- **Для производительности** — генерируйте PDF в фоне (DispatchQueue.global)  
- **Для [[SwiftUI]]** — создавайте PDF в `Task` и показывайте через `ShareLink` / [[UIActivityViewController]]  
- **Документируйте** — пишите комментарий «UIGraphicsPDFRenderer — генерация A4-документа с текстом и таблицей (многостраничный)»

**Короткий итог 2026**:
> UIGraphicsPDFRenderer — это **современный и единственный рекомендуемый** способ создавать PDF в iOS.  
> В 2026 году:  
> - заменяет устаревший CGPDFContext  
> - ключевой метод — `pdfData { context in ... }`  
> - идеально для: счетов, отчётов, экспорта документов, скриншотов в PDF  
> - поддерживает многостраничность, векторный рендеринг, правильные цвета  
