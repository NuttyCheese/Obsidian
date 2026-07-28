#WKContentWorld #WebKit #iOS #Swift #JavaScript #Security #Isolation #Scripting #iOS14 #WKWebView

---
**(изолированная среда выполнения JavaScript / защита от конфликтов скриптов)**

**WKContentWorld** — это класс из фреймворка **[[WebKit]]**, введённый в **iOS 14**, который предоставляет **изолированную область выполнения (namespace)** для JavaScript-кода в `WKWebView`. Он позволяет выполнять пользовательские скрипты, обработчики сообщений и инспектировать DOM в среде, **отдельной** от JavaScript самой веб-страницы, предотвращая конфликты переменных и повышая безопасность приложения. 

**Ключевые особенности (важно в 2026):**
- Каждый мир (`WKContentWorld`) имеет **собственный глобальный объект (`window`)** и свою цепочку прототипов, но все миры разделяют один **DOM** .
- Это позволяет безопасно внедрять скрипты приложения, не боясь, что страница переопределит ваши функции или переменные, и наоборот — ваши скрипты не "засорят" глобальное пространство страницы .
- Код в изолированном мире **не может быть перехвачен или изменён** вредоносным JavaScript на странице, так как у них разные экземпляры встроенных объектов (`Array`, `Object`, `document.querySelector` и т.д.) .

---

### Типы WKContentWorld

Существует три способа получить или создать контент-мир: 

| Мир | Доступ | Назначение |
|-----|--------|------------|
| `.page` | Статическое свойство | Мир самой веб-страницы. Скрипты и обработчики здесь разделяют глобальную область видимости с JavaScript страницы. **Осторожно:** этот мир уязвим для переопределения со стороны страницы (XSS, вредоносный код). |
| `.defaultClient` | Статическое свойство | **Рекомендуемый мир** для вашего приложения. Это изолированный мир, созданный специально для использования клиентом (вашим приложением). |
| `.world(withName:)` | Статический метод | Создаёт **пользовательский изолированный мир** с уникальным именем. Идеально подходит для изоляции друг от друга разных компонентов или расширений вашего приложения. |

---

### Схема взаимосвязей

```mermaid
graph TD
    A[WKWebView] --> B[DOM]
    
    subgraph PageWorld["Страница"]
        C[".page world"]
        C --> D[window.pageVar]
        C --> E[Array.prototype]
    end
    
    subgraph AppWorld["Ваше приложение"]
        F[".defaultClient world"]
        F --> G[window.appVar]
        F --> H[Array.prototype]
        
        I['.world name:"MyWorld"']
        I --> J[window.customVar]
        I --> K[Array.prototype]
    end
    
    B -.-> C
    B -.-> F
    B -.-> I
    
    D -.-> E
    G -.-> H
    J -.-> K
    
    style C fill:#ffcccc,stroke:#333,stroke-width:2px
    style F fill:#ccffcc,stroke:#333,stroke-width:2px
    style I fill:#ccddff,stroke:#333,stroke-width:2px
```

*Все миры имеют собственные глобальные объекты и прототипы, но разделяют один и тот же DOM.* 

---

### Основные API, поддерживающие WKContentWorld (iOS 14+)

Следующие API получили версии, принимающие `WKContentWorld` в качестве параметра: 

| API                                                                     | Назначение                                                                         |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `WKUserContentController.add(_:contentWorld:name:)`                     | Регистрирует обработчик сообщений в указанном мире.                                |
| `WKUserContentController.addScriptMessageHandler(_:contentWorld:name:)` | Регистрирует обработчик с поддержкой ответа ([[WKScriptMessageHandlerWithReply]]). |
| `WKUserScript(source:injectionTime:forMainFrameOnly:in:)`               | Внедряет пользовательский скрипт в указанный мир.                                  |
| `WKWebView.evaluateJavaScript(_:in:in:completionHandler:)`              | Выполняет JavaScript-код в указанном мире.                                         |
| `WKWebView.callAsyncJavaScript(_:arguments:in:in:completionHandler:)`   | Выполняет асинхронную JS-функцию с возвратом Promise в указанном мире.             |

> **Важно:** Варианты этих методов без параметра `in: contentWorld:` по умолчанию работают в **`.page`**, что **небезопасно** и подвержено конфликтам. 

---

### Примеры (от простого к сложному)

#### 1. Базовое использование: выполнение скрипта в изолированном мире
```swift
import UIKit
import WebKit

class ViewController: UIViewController {
    var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()
        webView = WKWebView(frame: view.bounds)
        view.addSubview(webView)
        
        // 1. Получаем изолированный мир приложения
        let appWorld = WKContentWorld.defaultClient
        
        // 2. Выполняем скрипт в этом мире
        webView.evaluateJavaScript("window.myAppSecret = 'VerySecret'", 
                                   in: nil, 
                                   in: appWorld) { result in
            print("Секретная переменная установлена в изолированном мире")
        }
        
        // 3. Проверяем, что скрипт НЕ виден в мире страницы
        webView.evaluateJavaScript("typeof window.myAppSecret") { result, error in
            // Выведет "undefined", так как переменная не в .page мире
            print("Переменная на странице: \(result ?? "nil")")
        }
    }
}
```

#### 2. Изоляция скрипта, читающего DOM, от переопределения на странице
Этот пример показывает, как изолированный мир защищает ваш инспектирующий код от подмены методов страницей. Даже если страница переопределит `document.getElementById`, ваш скрипт в изолированном мире получит оригинальный, неизменённый метод. 

```swift
// 1. Создаём изолированный мир
let appWorld = WKContentWorld.world(withName: "InspectionWorld")

// 2. Выполняем скрипт для чтения данных из DOM, который НЕ МОЖЕТ БЫТЬ ПЕРЕХВАТЧЕН
webView.evaluateJavaScript("document.getElementById('userBalance').textContent",
                           in: nil,
                           in: appWorld) { result in
    switch result {
    case .success(let value):
        // Обрабатываем значение. Оно получено через НЕИЗМЕНЁННУЮ функцию.
        print("Баланс пользователя: \(value)")
    case .failure(let error):
        print("Ошибка: \(error)")
    }
}
```

#### 3. Регистрация обработчика сообщений в изолированном мире (безопасный мост)
Вместо глобального обработчика в `.page` мире, регистрируем его в мире `.defaultClient`. Это предотвращает попытки страницы перехватить сообщение или выдать себя за ваше приложение. 

```swift
import UIKit
import WebKit

class SecureBridgeViewController: UIViewController, WKScriptMessageHandler {
    var webView: WKWebView!
    let appWorld = WKContentWorld.defaultClient

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let config = WKWebViewConfiguration()
        let contentController = WKUserContentController()
        
        // Регистрируем обработчик В ИЗОЛИРОВАННОМ МИРЕ приложения
        contentController.addScriptMessageHandler(self, 
                                                  contentWorld: appWorld, 
                                                  name: "secureBridge")
        
        config.userContentController = contentController
        webView = WKWebView(frame: view.bounds, configuration: config)
        view.addSubview(webView)
        
        // Внедряем скрипт, который отправляет сообщение
        let script = WKUserScript(source: """
            document.addEventListener('click', () => {
                window.webkit.messageHandlers.secureBridge.postMessage('Clicked!');
            });
        """, injectionTime: .atDocumentStart, 
        forMainFrameOnly: true, 
        in: appWorld) // Внедряем скрипт в ТОТ ЖЕ изолированный мир
        contentController.addUserScript(script)
    }
    
    func userContentController(_ userContentController: WKUserContentController, 
                               didReceive message: WKScriptMessage) {
        print("Получено безопасное сообщение: \(message.body)")
        // Этот обработчик не может быть перехвачен страницей
    }
    
    deinit {
        webView?.configuration.userContentController.removeScriptMessageHandler(forName: "secureBridge", 
                                                                               contentWorld: appWorld)
    }
}
```

---

### Важные нюансы 2026 года

> [!warning]
> **Совместный DOM:** Несмотря на изоляцию глобальных объектов и прототипов, все миры работают с **одним и тем же DOM-деревом**. Если вы измените свойство элемента в одном мире, это изменение будет видно во всех остальных мирах и на самой странице. 
>
> **Жизненный цикл:** `WKContentWorld` является пространством имён (namespace) и **не сохраняет данные** между разными `WKWebView` или после перезагрузки страницы. Переменные из предыдущей страницы будут утеряны при навигации. 
>
> **Безопасность:** Использование `.page` мира для ваших скриптов и обработчиков считается небезопасным, так как код страницы может переопределить глобальные объекты и перехватить данные. Это является потенциальной уязвимостью, проверяемой в OWASP MASVS. 

---

### Лучшие практики 2026

1.  **Всегда используйте `.defaultClient` или кастомный `.world(withName:)`** для выполнения вашего JavaScript-кода и регистрации обработчиков. 
2.  **Изолируйте обработчики сообщений (`WKScriptMessageHandler`)** от мира страницы, чтобы предотвратить их перехват. 
3.  **Создавайте отдельные кастомные миры** для разных компонентов приложения (например, "AuthWorld", "AnalyticsWorld") для дополнительной изоляции. 
4.  **Всегда удаляйте обработчики сообщений** (`removeScriptMessageHandler(forName:contentWorld:)`) в `deinit`, используя тот же мир, в котором они были зарегистрированы. 
5.  **Помните о совместном DOM** — даже в изолированном мире вы читаете и изменяете реальный DOM страницы, поэтому всегда валидируйте данные, полученные из него.

---

### Связь с другими темами

- [[WKUserContentController]] — управление обработчиками и скриптами
- [[WKScriptMessageHandler]] — обработка сообщений (устаревший вариант)
- [[WKScriptMessageHandlerWithReply]] — обработка с ответом
- [[WKUserScript]] — внедрение скриптов
- [[WKWebView]] — выполнение JS через `evaluateJavaScript(_:in:in:completionHandler:)`