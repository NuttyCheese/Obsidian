**`WKWebView`** — это основной и рекомендуемый компонент в iOS/macOS для отображения веб-контента внутри приложения (замена устаревшему `UIWebView`).

Он входит в фреймворк **WebKit** и представляет собой мощный, производительный и безопасный браузерный движок, основанный на **Safari**.

### Ключевые преимущества WKWebView (актуально на 2026 год)

| Характеристика                     | Описание                                                                 | Важные замечания 2026 |
|------------------------------------|--------------------------------------------------------------------------|------------------------|
| **Производительность**             | Использует тот же движок, что и Safari (WebKit)                          | Быстрее и экономичнее `UIWebView` в 10+ раз |
| **Поддержка современных стандартов**| HTML5, CSS3, JavaScript (ES2023+), WebAssembly, WebRTC, WebGL и т.д.     | Полная поддержка PWA и современных веб-приложений |
| **Безопасность**                   | Sandbox, App Transport Security, Content Blocker, Intelligent Tracking Prevention | Строгие требования к ATS и HTTPS |
| **JavaScript ↔ Native**            | `evaluateJavaScript`, `WKScriptMessageHandler`, `WKUserScript`           | Двусторонняя коммуникация |
| **Навигация и история**            | `goBack`, `goForward`, `reload`, `navigationDelegate`                    | Полный контроль навигации |
| **Конфигурация**                   | `WKWebViewConfiguration` — всё настраивается до создания                | `WKPreferences`, `WKWebsiteDataStore`, `WKProcessPool` |
| **Поддержка**                      | iOS 8+, macOS 10.10+, watchOS 4+, tvOS 9+                                | Полностью поддерживается в iOS 18+ |

### Минимальный рабочий пример (современный стиль 2026)

```swift
import UIKit
import WebKit

@MainActor
final class WebViewController: UIViewController, WKNavigationDelegate {
    
    private var webView: WKWebView!
    
    override func loadView() {
        let config = WKWebViewConfiguration()
        // config.userContentController.add(self, name: "nativeBridge") — если нужна JS ↔ Native
        
        webView = WKWebView(frame: .zero, configuration: config)
        webView.navigationDelegate = self
        webView.allowsBackForwardNavigationGestures = true
        view = webView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        if let url = URL(string: "https://www.example.com") {
            let request = URLRequest(url: url)
            webView.load(request)
        }
    }
    
    // Делегат: страница начала загрузки
    func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
        // Показать индикатор загрузки
        UIApplication.shared.isNetworkActivityIndicatorVisible = true
    }
    
    // Делегат: страница загрузилась
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        UIApplication.shared.isNetworkActivityIndicatorVisible = false
        title = webView.title  // обновляем заголовок навигации
    }
    
    // Делегат: ошибка загрузки
    func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
        UIApplication.shared.isNetworkActivityIndicatorVisible = false
        print("Ошибка загрузки:", error.localizedDescription)
        
        let alert = UIAlertController(title: "Ошибка", message: error.localizedDescription, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    // Делегат: решение о навигации (можно блокировать/перенаправлять)
    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction,
                 decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        
        if let url = navigationAction.request.url, url.scheme == "myapp" {
            // Обработка deep link / custom scheme
            print("Custom scheme:", url)
            decisionHandler(.cancel)
            return
        }
        
        decisionHandler(.allow)
    }
}
```

### Как добавить JavaScript ↔ Native взаимодействие (очень популярно)

```swift
// В конфигурации
let config = WKWebViewConfiguration()
let userScript = WKUserScript(source: "window.webkit.messageHandlers.nativeBridge.postMessage('Hello from JS')",
                             injectionTime: .atDocumentEnd,
                             forMainFrameOnly: true)
config.userContentController.addUserScript(userScript)
config.userContentController.add(self, name: "nativeBridge")

// В делегате
extension WebViewController: WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController,
                               didReceive message: WKScriptMessage) {
        if message.name == "nativeBridge" {
            print("Сообщение из JavaScript:", message.body)
        }
    }
}
```

### Лучшие практики WKWebView в Swift 2026

- **Всегда** устанавливайте `navigationDelegate` и обрабатывайте `didFailProvisionalNavigation`  
- **Используйте** `WKWebViewConfiguration` для настройки до создания экземпляра  
- **Для JavaScript** — `evaluateJavaScript` (async/await) или `WKScriptMessageHandler`  
- **Для кастомных схем** (`myapp://`) — перехватывайте в `decidePolicyFor navigationAction`  
- **Для оффлайн-контента** — используйте `WKWebsiteDataStore` + `loadHTMLString`  
- **В SwiftUI** — оборачивайте в `UIViewRepresentable` + `Coordinator` (делегат)  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для WebKit  
- **Документируйте** — пишите комментарий «WKWebView — отображение веб-контента с обработкой навигации и ошибок»

**Короткий итог 2026**:
> `WKWebView` — это **нативный и мощный** веб-браузер внутри вашего приложения.  
> В 2026 году:  
> - главный делегат — `navigationDelegate`  
> - ключевые методы — `didStartProvisionalNavigation`, `didFinish`, `didFailProvisionalNavigation`  
> - для JS ↔ Native — `WKScriptMessageHandler` или `evaluateJavaScript`  
> - всегда проверяйте разрешения и обрабатывайте ошибки  
> Это **единственный** рекомендуемый способ показывать веб-контент в iOS-приложениях.

Удачи с гибкими и производительными веб-вью в твоём проекте! 🌐