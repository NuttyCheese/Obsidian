#network #Swift 
**Deep link** — это [[URL]], который открывает **конкретный экран или контент внутри приложения**, а не просто запускает его.

### Два основных способа в iOS (2026)

| Тип                 | Технология                        | Преимущества                          | Недостатки                                  | Когда использовать                  |
| ------------------- | --------------------------------- | ------------------------------------- | ------------------------------------------- | ----------------------------------- |
| Custom URL Scheme   | `myapp://profile/123`             | Простота, работает без интернета      | Нет подтверждения владения доменом          | Быстрые ссылки, внутренние переходы |
| [[Universal Link]]  | `https://example.com/profile/123` | Безопасно, открывается в Safari → app | Требует сервер + apple-app-site-association | Основной способ в 2026 году         |
| App Links (Android) | —                                 | —                                     | —                                           | Кросс-платформенные проекты         |

### 1. Custom URL Schemes — самый простой способ

#### Шаг 1: Регистрация схемы в Info.plist

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
        <key>CFBundleURLName</key>
        <string>com.example.myapp</string>
    </dict>
</array>
```

#### Шаг 2: Обработка в [[AppDelegate]] / [[SceneDelegate]]

```swift
// SceneDelegate (iOS 13+)
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    handleDeepLink(url: url)
}

// AppDelegate (для совместимости)
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    handleDeepLink(url: url)
    return true
}

func handleDeepLink(url: URL) {
    guard url.scheme == "myapp" else { return }
    
    switch url.host {
    case "profile":
        if let userID = url.queryParameters?["id"] {
            navigateToProfile(userID: userID)
        }
    case "settings":
        navigateToSettings()
    case "product":
        if let productID = url.queryParameters?["id"] {
            navigateToProduct(id: productID)
        }
    default:
        break
    }
}

extension URL {
    var queryParameters: [String: String]? {
        URLComponents(url: self, resolvingAgainstBaseURL: false)?
            .queryItems?
            .reduce(into: [String: String]()) { $0[$1.name] = $1.value }
    }
}
```

#### Шаг 3: Навигация ([[UIKit]])

```swift
func navigateToProfile(userID: String) {
    guard let nav = window?.rootViewController as? UINavigationController else { return }
    
    if let profileVC = nav.viewControllers.first(where: { $0 is ProfileViewController }) as? ProfileViewController {
        profileVC.userID = userID
        nav.popToViewController(profileVC, animated: true)
    } else {
        let profileVC = ProfileViewController()
        profileVC.userID = userID
        nav.pushViewController(profileVC, animated: true)
    }
}
```

### 2. [[Universal Link]] — современный и рекомендуемый способ

#### Шаг 1: Настройка Associated Domains в [[Xcode]]

- В Signing & Capabilities → + Capability → Associated Domains
- Добавить: `applinks:example.com`

#### Шаг 2: Файл apple-app-site-association на сервере

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.example.myapp",
        "paths": ["/profile/*", "/product/*", "/settings"]
      }
    ]
  }
}
```

Расположение: `https://example.com/.well-known/apple-app-site-association`

#### Шаг 3: Обработка в SceneDelegate

```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    
    handleUniversalLink(url: url)
}

func handleUniversalLink(url: URL) {
    switch url.path {
    case let path where path.hasPrefix("/profile/"):
        let userID = path.components(separatedBy: "/").last
        navigateToProfile(userID: userID)
    case "/settings":
        navigateToSettings()
    default:
        break
    }
}
```

### 3. [[Deep Link]] в [[SwiftUI]] (NavigationStack + onOpenURL)

```swift
@main
struct MyApp: App {
    @State private var path = NavigationPath()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $path) {
                HomeView()
                    .onOpenURL { url in
                        handleDeepLink(url: url)
                    }
            }
            .navigationDestination(for: Route.self) { route in
                switch route {
                case .profile(let id): ProfileView(userID: id)
                case .settings: SettingsView()
                }
            }
        }
    }
    
    private func handleDeepLink(url: URL) {
        guard url.scheme == "myapp" || url.host == "example.com" else { return }
        
        switch url.host {
        case "profile":
            if let id = url.queryParameters?["id"] {
                path.append(Route.profile(id: id))
            }
        case "settings":
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

### 4. Дополнительные реальные примеры

#### Пример 5 — Deep link с авторизацией (OAuth callback)

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url,
          url.scheme == "myapp",
          url.host == "oauth-callback" else { return }
    
    // Обработка кода авторизации
    if let code = url.queryParameters?["code"] {
        Task {
            try await AuthService.shared.exchangeCodeForToken(code)
        }
    }
}
```

#### Пример 6 — Универсальная обработка deep link с роутингом

```swift
class DeepLinkRouter {
    static let shared = DeepLinkRouter()
    weak var navigationController: UINavigationController?
    
    func handle(url: URL) {
        guard url.scheme == "myapp" || url.host == "example.com" else { return }
        
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
        let path = components?.path ?? ""
        
        switch path {
        case "/profile":
            if let id = components?.queryItems?.first(where: { $0.name == "id" })?.value {
                showProfile(id: id)
            }
        case "/cart":
            showCart()
        case "/product":
            if let id = components?.queryItems?.first(where: { $0.name == "id" })?.value {
                showProduct(id: id)
            }
        default:
            break
        }
    }
    
    private func showProfile(id: String) {
        let vc = ProfileViewController(userID: id)
        navigationController?.pushViewController(vc, animated: true)
    }
    
    // ...
}
```

### Лучшие практики deep linking 2026

- Используй **Universal Links** как основной способ
- Custom URL Scheme — только как запасной вариант
- Всегда обрабатывай URL в `scene(_:openURLContexts:)` и `scene(_:continue:)`
- Для SwiftUI — `onOpenURL` + `NavigationStack` / `NavigationPath`
- Для UIKit — централизованный роутер (Coordinator / Router)
- Тестируй через **xcrun simctl openurl** и **Safari → Universal Links**
- Добавляй **отложенную обработку** (если приложение не запущено)

**Короткое правило**:
> «Universal Links — основной способ в 2026.  
> Custom scheme — только для fallback.  
> Централизуй обработку в одном месте ([[Clean Swift (VIP) Architecture#**1. Взаимодействие компонентов (VIP-цикл)**|Router]] / [[Coordinator]]).  
> Всегда проверяй scheme/host/path/query.»
