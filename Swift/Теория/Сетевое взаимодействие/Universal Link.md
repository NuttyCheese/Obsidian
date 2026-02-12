#network #Swift 
**Universal Links** — это механизм Apple, позволяющий **ссылкам на веб-сайте** открывать **конкретный контент внутри приложения**, если оно установлено, или в Safari, если приложения нет.

Это **современный и рекомендуемый** способ глубоких ссылок ([[deep link]]) в [[iOS]].

### Почему Universal Links лучше Custom [[URL]] Schemes

| Характеристика                     | Custom URL Scheme (`myapp://`) | Universal Links (`https://`)          |
|------------------------------------|--------------------------------|----------------------------------------|
| Работает без установки приложения  | Нет                            | Да (открывается в Safari)              |
| Безопасность (верификация домена)  | Нет                            | Да (apple-app-site-association)        |
| Бесшовный переход                  | Плохо (переключение приложений)| Отлично (Safari → app без перезагрузки)|
| SEO и индексация                   | Нет                            | Да (Google, Apple Search)              |
| Поддержка в 2026 году              | Устаревает                     | Рекомендуемый и основной способ         |
| Обработка в фоне                   | Сложно                         | Легко (userActivity)                   |

### Пошаговая настройка Universal Links (2026)

#### Шаг 1: Включение Associated Domains в [[Xcode]]

1. Открой проект → Signing & Capabilities  
2. Нажми «+ Capability» → Associated Domains  
3. Добавь домен с префиксом `applinks:`

Примеры:
```
applinks:example.com
applinks:api.example.com
applinks:links.myapp.ru
```

#### Шаг 2: Создание и размещение файла `apple-app-site-association`

Файл должен лежать по адресу:

```
https://example.com/.well-known/apple-app-site-association
или
https://example.com/apple-app-site-association
```

Содержимое (минимальное, 2026):

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appIDs": [ "TEAMID.com.example.myapp" ],
        "components": [
          {
            "/": "/profile/*",
            "comment": "Профиль пользователя"
          },
          {
            "/": "/product/*",
            "comment": "Страница товара"
          },
          {
            "/": "/settings",
            "comment": "Настройки"
          }
        ]
      }
    ]
  }
}
```

**Важно**:
- `TEAMID` — ваш Apple Developer Team ID (10 символов)
- `com.example.myapp` — Bundle ID приложения
- `paths` или `components` — поддерживаемые пути (можно использовать `*`, `?`, `#`)
- Файл **без расширения**, MIME-тип — `application/json`
- Сервер должен отдавать его по **[[HTTPS]]** с валидным сертификатом

#### Шаг 3: Проверка файла (инструменты 2026)

- Браузер: открой `https://example.com/.well-known/apple-app-site-association`
- Apple-валидатор: https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html (или Branch.io AASA Validator)
- Команда в терминале:
  ```bash
  curl -I https://example.com/.well-known/apple-app-site-association
  ```

Ожидаемый ответ:
```
HTTP/2 200 
Content-Type: application/json
```

#### Шаг 4: Обработка Universal Links в коде

##### Вариант A — [[UIKit]] + [[SceneDelegate]] (iOS 13+)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
              let incomingURL = userActivity.webpageURL else {
            return
        }
        
        handleUniversalLink(url: incomingURL)
    }
    
    private func handleUniversalLink(url: URL) {
        // Разбор пути и параметров
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
        let path = components?.path ?? ""
        
        switch path {
        case let p where p.hasPrefix("/profile/"):
            let userID = p.components(separatedBy: "/").last
            openProfile(userID: userID)
            
        case "/settings":
            openSettings()
            
        case let p where p.hasPrefix("/product/"):
            let productID = p.components(separatedBy: "/").last
            openProduct(id: productID)
            
        default:
            // fallback — открыть главную
            openHome()
        }
    }
    
    private func openProfile(userID: String?) {
        guard let nav = window?.rootViewController as? UINavigationController else { return }
        
        let profileVC = ProfileViewController()
        profileVC.userID = userID
        nav.pushViewController(profileVC, animated: true)
    }
    
    // Другие методы аналогично...
}
```

##### Вариант B — [[SwiftUI]] + NavigationStack (iOS 16+)

```swift
@main
struct MyApp: App {
    @State private var path = NavigationPath()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $path) {
                HomeView()
                    .onOpenURL { url in
                        handleUniversalLink(url: url)
                    }
            }
            .navigationDestination(for: Route.self) { route in
                switch route {
                case .profile(let id):      ProfileView(userID: id)
                case .settings:             SettingsView()
                case .product(let id):      ProductView(productID: id)
                }
            }
        }
    }
    
    private func handleUniversalLink(url: URL) {
        guard url.host == "example.com" else { return }
        
        let path = url.path
        
        switch path {
        case let p where p.hasPrefix("/profile/"):
            let userID = p.components(separatedBy: "/").last
            if let id = userID {
                path.append(Route.profile(id: id))
            }
            
        case "/settings":
            path.append(Route.settings)
            
        default:
            break
        }
    }
}

enum Route: Hashable {
    case profile(id: String)
    case settings
}
```

### Таблица: Universal Links vs Custom URL Schemes (2026)

| Характеристика                     | Universal Links (`https://`)          | Custom URL Scheme (`myapp://`) |
|------------------------------------|----------------------------------------|--------------------------------|
| Открывается без приложения         | Да (в Safari)                          | Нет                            |
| Верификация владения доменом       | Да (AASA-файл)                         | Нет                            |
| Бесшовный переход                  | Отлично (Safari → app)                 | Средне (переключение приложений)|
| SEO и индексация                   | Да                                     | Нет                            |
| Поддержка в Safari                 | Да                                     | Да                             |
| Обработка в фоне                   | Да (`userActivity`)                    | Да (`open url`)                |
| Рекомендация Apple 2026            | Основной способ                        | Fallback / legacy              |

### Типичные ошибки и как их избежать

| Ошибка                                      | Симптомы                           | Как исправить                                           |
| ------------------------------------------- | ---------------------------------- | ------------------------------------------------------- |
| Неправильный путь к AASA-файлу              | Universal Links не открываются     | Разместить по `/.well-known/apple-app-site-association` |
| Неверный `appID` в AASA                     | Ссылка открывается только в Safari | Проверить TEAMID + Bundle ID                            |
| Нет `applinks:` в Associated Domains        | Ничего не происходит               | Добавить в Signing & Capabilities                       |
| Не вызван `invalidate()` у наблюдателей     | Утечка памяти                      | Использовать `NSKeyValueObservation` с `invalidate()`   |
| Обработка только в AppDelegate              | Не работает на iOS 13+             | Добавить обработку в [[SceneDelegate]]                  |
| Нет проверки `activityType == .browsingWeb` | Обрабатываются не те активности    | Всегда проверять                                        |

### Инструменты для отладки Universal Links (2026)

- **xcrun simctl openurl booted https://example.com/profile/123** — тест в симуляторе
- **Branch.io AASA Validator** — проверка файла
- **Apple App Search API Validation Tool** — официальный валидатор
- **Safari → Developer → Web Inspector** — отладка перехода
- **Console.app** — логи `swcd` (Shared Web Credentials daemon)

**Короткое правило 2026**:
> «Universal Links — **единственный рекомендуемый** способ глубоких ссылок в iOS.  
> Custom scheme — только fallback для старых устройств или внутренних переходов.  
> Всегда проверяй AASA-файл, Associated Domains и обработку в SceneDelegate.»
