**WKNavigationDelegate** — это протокол в фреймворке **[[WebKit]]**, который отвечает за отслеживание и управление процессом навигации в [[WKWebView]].

Он позволяет:

- решать, разрешать ли загрузку определённого URL
- реагировать на начало/окончание загрузки страницы
- обрабатывать ошибки загрузки
- перехватывать переходы по ссылкам (в т.ч. target="_blank", JavaScript-редиректы, mailto:, tel:, app-links)
- отслеживать прогресс загрузки, получение заголовков, сертификатов и т.д.

Это **самый важный** протокол для любого приложения, которое использует `WKWebView`.

### Основные методы протокола (актуальные на 2026 год)

| Метод делегата                                                 | Когда вызывается                                                            | Что обычно делают внутри метода                                               | Обязателен?       | Частота использования |
| -------------------------------------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------- | --------------------- |
| `webView(_:decidePolicyFor:decisionHandler:)`                  | Перед каждой навигацией (включая редиректы, клики по ссылкам, JS-редиректы) | Самый важный метод: решать — разрешить/отменить/открыть в Safari/в новом окне | Да (почти всегда) | ★★★★★                 |
| `webView(_:didStartProvisionalNavigation:)`                    | Начало загрузки страницы (запрос отправлен, но контент ещё не получен)      | Показать индикатор загрузки, обновить UI                                      | Нет               | ★★★★☆                 |
| `webView(_:didReceiveServerRedirectForProvisionalNavigation:)` | Сервер вернул [[HTTP]] 3xx редирект                                         | Логирование, аналитика, иногда отмена                                         | Нет               | ★★★☆☆                 |
| `webView(_:didCommit:)`                                        | Получен первый байт контента (навигация подтверждена)                       | Скрыть индикатор "загрузка началась", начать показывать прогресс              | Нет               | ★★★★☆                 |
| `webView(_:didFinish:)`                                        | Страница полностью загружена (все ресурсы, включая картинки/скрипты)        | Скрыть индикатор загрузки, включить взаимодействие, выполнить JS              | Нет               | ★★★★★                 |
| `webView(_:didFail:withError:)`                                | Ошибка загрузки (сервер не найден, SSL-ошибка, таймаут и т.д.)              | Показать ошибку, кнопку "Повторить", аналитика                                | Нет               | ★★★★☆                 |
| `webView(_:didFailProvisionalNavigation:withError:)`           | Ошибка на этапе provisional (до didCommit)                                  | Показать ошибку "Не удалось загрузить страницу"                               | Нет               | ★★★☆☆                 |
| `webView(_:didReceive:challenge:completionHandler:)`           | Требуется обработка SSL/TLS-сертификата (self-signed, expired и т.д.)       | Разрешить/отклонить сертификат (самый частый кейс — корпоративные прокси)     | Нет               | ★★★☆☆                 |
| `webViewWebContentProcessDidTerminate(_:)`                     | Процесс WebContent упал (краш WKWebView)                                    | Перезагрузить страницу (`reload()`), показать ошибку                          | Нет               | ★★☆☆☆                 |

### Самый полный и рекомендуемый паттерн 2026 года

```swift
class WebViewController: UIViewController, WKNavigationDelegate {
    
    private var webView: WKWebView!
    
    override func loadView() {
        let config = WKWebViewConfiguration()
        webView = WKWebView(frame: .zero, configuration: config)
        webView.navigationDelegate = self
        view = webView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Веб-сайт"
        
        if let url = URL(string: "https://example.com") {
            webView.load(URLRequest(url: url))
        }
    }
    
    // Самый важный метод — контроль навигации
    func webView(_ webView: WKWebView,
                 decidePolicyFor navigationAction: WKNavigationAction,
                 decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        
        guard let url = navigationAction.request.url else {
            decisionHandler(.cancel)
            return
        }
        
        // Разрешаем только HTTPS и наши домены
        if url.scheme != "https" {
            decisionHandler(.cancel)
            return
        }
        
        // Открываем внешние ссылки в Safari
        if !url.absoluteString.hasPrefix("https://example.com") {
            UIApplication.shared.open(url)
            decisionHandler(.cancel)
            return
        }
        
        // Разрешаем все остальные переходы
        decisionHandler(.allow)
    }
    
    func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
        // Показываем индикатор загрузки
        UIApplication.shared.isNetworkActivityIndicatorVisible = true
    }
    
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        // Скрываем индикатор
        UIApplication.shared.isNetworkActivityIndicatorVisible = false
        
        // Можно выполнить JS
        webView.evaluateJavaScript("document.title") { title, _ in
            self.title = title as? String ?? "Веб-сайт"
        }
    }
    
    func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
        UIApplication.shared.isNetworkActivityIndicatorVisible = false
        
        let alert = UIAlertController(title: "Ошибка загрузки", message: error.localizedDescription, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    // Обработка SSL-сертификатов (самый частый кейс в корпоративных приложениях)
    func webView(_ webView: WKWebView,
                 didReceive challenge: URLAuthenticationChallenge,
                 completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
           let serverTrust = challenge.protectionSpace.serverTrust {
            
            // Для self-signed сертификатов (например, dev-сервер)
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            completionHandler(.performDefaultHandling, nil)
        }
    }
}
```

### Лучшие практики UISearchResultsUpdating в 2026 году

- **Всегда** реализуйте `decidePolicyFor` — это единственный способ контролировать, какие ссылки открывать  
- **Для внешних ссылок** — открывайте через `UIApplication.shared.open(url)`  
- **Для mailto:, tel:, sms:** — тоже через `open(url)` или кастомную обработку  
- **Для JavaScript-редиректов** — они тоже попадают в `decidePolicyFor`  
- **Для ошибок SSL** — реализуйте `didReceive challenge` (особенно в корпоративных приложениях)  
- **Для индикатора сети** — используйте `didStartProvisionalNavigation` / `didFinish`  
- **Для [[SwiftUI]]** — используйте `WebView` через `UIViewRepresentable` + делегат  
- **Документируйте** — пишите комментарий «WKNavigationDelegate — контроль навигации, обработка внешних ссылок и SSL-сертификатов»

**Короткий итог 2026**:
> `WKNavigationDelegate` — это **главный протокол** для управления навигацией в `WKWebView`.  
> В 2026 году:  
> - ключевой метод — `decidePolicyFor navigationAction` (разрешать/запрещать ссылки)  
> - остальные методы — для отслеживания прогресса, ошибок, SSL  
> - это **обязательный** делегат для любого приложения с веб-контентом  
> - без него `WKWebView` работает, но без контроля и обработки ошибок  
