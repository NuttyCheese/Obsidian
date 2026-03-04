#uiwebview #uikit #deprecated #webview #ios #swift #legacy #web-content #ios-12 #ios-8

---
**(устаревший веб-вью / веб-браузер в приложении)**

**UIWebView** — это **устаревший** (deprecated с iOS 8, полностью удалён в iOS 12) класс в [[UIKit]], который позволял отображать веб-контент (HTML, сайты, локальные файлы .html) прямо внутри приложения.

Он был основным способом показывать веб-страницы в приложениях до появления **[[WKWebView]]** (iOS 8, 2014).

**Важный статус в 2026 году:**

- **Полностью удалён** из iOS начиная с iOS 12 (2018)
- При попытке использовать в проекте с deployment target iOS 12+ → **ошибка компиляции**
- App Store **отклоняет** приложения с UIWebView с 2020 года (App Review Guideline 2.5.6)
- Apple официально рекомендует **только WKWebView**

Если вы видите `UIWebView` в старом коде 2025–2026 годов — это **legacy-код**, который нужно срочно мигрировать.

### Почему UIWebView устарел и опасен

| Проблема                                 | Почему критично в 2026 году                           | Последствия                           |
| ---------------------------------------- | ----------------------------------------------------- | ------------------------------------- |
| Нет поддержки современных веб-стандартов | Нет JavaScript ES6+, WebGL 2, WebRTC, Service Workers | Сайты ломаются, медленно работают     |
| Уязвимости безопасности                  | Не получает обновлений безопасности с 2018 года       | Возможны атаки через [[WebKit]]-багги |
| Нет производительности                   | Использует старый WebKit-движок (до 2014)             | Медленно, высокий расход батареи      |
| Нет App Store-одобрения                  | Guideline 2.5.6 — запрет на использование UIWebView   | Отказ в публикации                    |
| Нет поддержки iOS 12+                    | Класс отсутствует в SDK                               | Ошибка компиляции                     |

### Как выглядит старый код с UIWebView (legacy)

```swift
// Никогда не используйте это в 2026 году!
import UIKit

class LegacyWebViewController: UIViewController {
    
    private var webView: UIWebView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        webView = UIWebView(frame: view.bounds)
        webView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        webView.scalesPageToFit = true
        view.addSubview(webView)
        
        if let url = URL(string: "https://example.com") {
            webView.loadRequest(URLRequest(url: url))
        }
    }
}
```

### Миграция с UIWebView на WKWebView (обязательно в 2026)

**WKWebView** — это современная, безопасная и производительная замена.

**Быстрая миграция (самый частый случай):**

```swift
import UIKit
import WebKit

class ModernWebViewController: UIViewController, WKNavigationDelegate {
    
    private var webView: WKWebView!
    
    override func loadView() {
        let config = WKWebViewConfiguration()
        // Можно добавить user scripts, handlers и т.д.
        webView = WKWebView(frame: .zero, configuration: config)
        webView.navigationDelegate = self
        view = webView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        if let url = URL(string: "https://example.com") {
            let request = URLRequest(url: url)
            webView.load(request)
        }
    }
    
    // Опционально: отслеживание загрузки
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        title = webView.title
    }
    
    func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error) {
        print("Ошибка загрузки: \(error)")
    }
}
```

### Ключевые различия UIWebView vs WKWebView

| Характеристика             | UIWebView (устарел)    | WKWebView (рекомендуется)                          | Почему важно в 2026        |
| -------------------------- | ---------------------- | -------------------------------------------------- | -------------------------- |
| Поддержка iOS              | iOS 2 – iOS 11         | iOS 8+ (полная поддержка)                          | UIWebView не компилируется |
| Производительность         | Низкая (старый движок) | Высокая (многопроцессный WebKit)                   | Быстрее на 2–5×            |
| Безопасность               | Устаревшие патчи       | Регулярные обновления безопасности                 | Нет уязвимостей            |
| JavaScript / Web-стандарты | До ES5                 | ES2023+, WebGL 2, WebRTC, Service Workers          | Современные сайты работают |
| Background execution       | Ограничено             | Полная поддержка (background fetch)                | Фоновые задачи             |
| App Store                  | Запрещено с 2020       | Разрешено                                          | Не отклонят                |
| Кастомизация               | Ограничена             | [[WKWebViewConfiguration]], user scripts, handlers | Гибкость                   |

### Лучшие практики миграции и использования в 2026 году

- **Никогда** не используйте UIWebView в новых проектах  
- **Мигрируйте** все старые экраны на WKWebView (обычно замена 1:1)  
- **Настройте конфигурацию** в `WKWebViewConfiguration` (javascriptEnabled, mediaPlaybackRequiresUserAction и т.д.)  
- **Добавьте WKNavigationDelegate** для отслеживания загрузки, ошибок, редиректов  
- **Используйте App Transport Security** (ATS) — добавьте исключения в Info.plist при необходимости  
- **Для локальных файлов** — используйте `loadFileURL(_:allowingReadAccessTo:)`  
- **Для JavaScript-взаимодействия** — используйте `evaluateJavaScript(_:completionHandler:)`  
- **Документируйте** — пишите комментарий:

```swift
/// Современный WKWebView вместо устаревшего UIWebView
private var webView: WKWebView!
```

**Короткий итог 2026**:
> **UIWebView** — **устаревший и запрещённый** способ показывать веб-контент в iOS-приложениях.  
> В 2026 году:  
> - полностью удалён из iOS 12+  
> - App Store отклоняет приложения с UIWebView  
> - **единственная замена** — **WKWebView**  
> - мигрируйте все старые экраны на WKWebView (замена почти 1:1)  
> - это **критическая** миграция для любого legacy-проекта  
