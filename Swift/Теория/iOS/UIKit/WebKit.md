**WebKit** — это мощный и открытый движок рендеринга веб-контента, который лежит в основе **Safari**, всех встроенных веб-представлений в iOS/macOS и многих других браузеров (Edge на iOS/macOS, Firefox на iOS до 2024–2025 и т.д.).

В контексте iOS-разработки в 2026 году WebKit = это практически синоним **WKWebView** и связанных с ним фреймворков.

### Ключевые компоненты WebKit на iOS/macOS (актуально на 2026 год)

| Компонент                  | Что это                                                                 | Когда использовать в 2026 | Самый частый сценарий |
|----------------------------|--------------------------------------------------------------------------|----------------------------|-----------------------|
| **WKWebView**              | Основной класс для отображения веб-контента в приложении                 | Всегда, когда нужен веб-контент внутри приложения | Встроенный браузер, OAuth, документация, гибридные экраны |
| **WKWebViewConfiguration** | Настройки WKWebView (JavaScript, cookies, media, processes и т.д.)       | Почти всегда перед созданием WKWebView | Отключение JS, включение PiP, кастомные schemes |
| **WKNavigationDelegate**   | Делегат для контроля навигации, редиректов, ошибок, SSL                 | Обязателен для любого серьёзного использования | Блокировка внешних ссылок, обработка ошибок, кастомные схемы |
| **WKUIDelegate**           | Делегат для UI-элементов (alert, confirm, prompt, новое окно)           | Если нужен alert/prompt или target="_blank" | Полноценный веб-опыт внутри приложения |
| **WKScriptMessageHandler** | Получение сообщений из JavaScript → нативный код                        | Гибридные приложения (Web → Native) | Вызов нативных функций из JS |
| **WKHTTPCookieStore**      | Управление cookies вручную                                              | OAuth, авторизация через WebView | Сохранение/удаление куки после логина |
| **WKWebsiteDataStore**     | Управление кэшем, cookies, localStorage и т.д.                           | Очистка данных, приватный режим | Инкогнито-режим в браузере |
| **WKProcessPool**          | Пул процессов WebContent (для ускорения и изоляции)                     | Редко, но критично при множестве WKWebView | Приложения с 10+ WebView на экране |

### Самый популярный и рекомендуемый паттерн 2026 года (WKWebView + делегаты)

```swift
import UIKit
import WebKit

class WebViewController: UIViewController, WKNavigationDelegate, WKUIDelegate {
    
    private var webView: WKWebView!
    
    override func loadView() {
        let config = WKWebViewConfiguration()
        
        // Самые частые настройки 2026
        config.allowsInlineMediaPlayback = true
        config.allowsPictureInPictureMediaPlayback = true
        config.mediaTypesRequiringUserActionForPlayback = [] // автоплей видео
        config.defaultWebpagePreferences.preferredContentMode = .mobile
        
        webView = WKWebView(frame: .zero, configuration: config)
        webView.navigationDelegate = self
        webView.uiDelegate = self
        view = webView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Документация"
        
        if let url = URL(string: "https://developer.apple.com/documentation") {
            webView.load(URLRequest(url: url))
        }
    }
    
    // MARK: - WKNavigationDelegate
    
    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        
        guard let url = navigationAction.request.url else {
            decisionHandler(.cancel)
            return
        }
        
        // Самые частые проверки 2026
        if url.scheme == "tel" || url.scheme == "mailto" || url.scheme == "sms" {
            UIApplication.shared.open(url)
            decisionHandler(.cancel)
            return
        }
        
        // Разрешаем только HTTPS и наши домены
        if url.scheme != "https" || !url.host!.hasSuffix("apple.com") {
            UIApplication.shared.open(url)
            decisionHandler(.cancel)
            return
        }
        
        decisionHandler(.allow)
    }
    
    func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
        let alert = UIAlertController(title: "Ошибка загрузки", message: error.localizedDescription, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "Повторить", style: .default) { _ in
            webView.reload()
        })
        alert.addAction(UIAlertAction(title: "OK", style: .cancel))
        present(alert, animated: true)
    }
    
    // MARK: - WKUIDelegate (для alert/confirm/prompt)
    
    func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
        let alert = UIAlertController(title: "Сообщение", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default) { _ in
            completionHandler()
        })
        present(alert, animated: true)
    }
}
```

### Тренды WebKit / WKWebView в 2026 году

- **JavaScript → Native** — всё больше гибридных приложений используют `WKScriptMessageHandler` + `window.webkit.messageHandlers`
- **Privacy & Tracking Prevention** — Apple усиливает ITP (Intelligent Tracking Prevention) → cookies и fingerprinting всё сложнее
- **Web Extensions** — поддержка расширений Safari в iOS 15+, но в WKWebView — только частично
- **WebAuthn / Passkeys** — нативная поддержка в WKWebView (Face ID / Touch ID для авторизации)
- **Web Push** — пока только в Safari, в WKWebView не поддерживается
- **Service Workers / PWA** — ограниченная поддержка в WKWebView
- **Performance** — WKWebView использует отдельный процесс WebContent → стабильность выросла

### Лучшие практики WKWebView в 2026 году

- **Всегда** задавайте `WKWebViewConfiguration` перед созданием
- **Обязательно** реализуйте `decidePolicyFor` — блокируйте внешние ссылки, tel/mailto и т.д.
- **Для JS → Native** — используйте `WKScriptMessageHandler` + `evaluateJavaScript`
- **Для очистки данных** — используйте `WKWebsiteDataStore.default.removeData(...)`
- **Для PiP / AirPlay** — включайте `allowsPictureInPictureMediaPlayback = true`
- **Для приватности** — используйте `WKWebsiteDataStore.nonPersistent()` для инкогнито-режима
- **Документируйте** — пишите комментарий:

```swift
/// WKWebView с контролем навигации: только HTTPS, внешние ссылки открываются в Safari
let config = WKWebViewConfiguration()
config.allowsInlineMediaPlayback = true
config.allowsPictureInPictureMediaPlayback = true
```

**Короткий итог 2026**:
> WebKit — это движок Safari и WKWebView, основной способ показывать веб-контент в iOS-приложениях.  
> В 2026 году:  
> - главный класс — **WKWebView**  
> - ключевые делегаты — **WKNavigationDelegate** (редиректы, ошибки) и **WKUIDelegate** (alert/prompt)  
> - обязательно контролируйте навигацию через `decidePolicyFor`  
> - это **единственный** официальный способ рендерить HTML/CSS/JS внутри приложения  

Удачи с безопасным и современным веб-контентом в твоём приложении! 🌐