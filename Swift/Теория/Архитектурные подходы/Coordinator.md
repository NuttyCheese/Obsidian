#coordinator #ios #architecture #navigation #flow_management #viewcontroller #modular #separation_of_concerns #routing
## 📘 Определение

**Coordinator** — паттерн в [[iOS]]-разработке, предназначенный для управления навигацией между экранами.  
Основная идея: **вынести логику переходов из ViewController**, чтобы сделать код более чистым, модульным и тестируемым.  
Часто используется с **[[UIKit]]**, но может применяться и в [[SwiftUI]] для управления NavigationStack или Flow.

---

## 🔹 Примеры кода

### 1. Простейший Coordinator с одним экраном

```swift
import UIKit

protocol Coordinator {
    func start()
}

class MainCoordinator: Coordinator {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        let vc = UIViewController()
        vc.view.backgroundColor = .white
        vc.title = "Main"
        navigationController.pushViewController(vc, animated: true)
    }
}
```

---

### 2. Coordinator с переходом на второй экран

```swift
class MainCoordinator: Coordinator {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        let vc = UIViewController()
        vc.view.backgroundColor = .white
        vc.title = "Main"
        vc.navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "Next",
            style: .plain,
            target: self,
            action: #selector(goNext)
        )
        navigationController.pushViewController(vc, animated: true)
    }

    @objc func goNext() {
        let secondVC = UIViewController()
        secondVC.view.backgroundColor = .lightGray
        secondVC.title = "Second"
        navigationController.pushViewController(secondVC, animated: true)
    }
}
```

---

### 3. Coordinator с несколькими экранами через методы

```swift
class AppCoordinator: Coordinator {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        showLogin()
    }

    private func showLogin() {
        let loginVC = UIViewController()
        loginVC.view.backgroundColor = .white
        loginVC.title = "Login"
        navigationController.pushViewController(loginVC, animated: true)
    }

    private func showHome() {
        let homeVC = UIViewController()
        homeVC.view.backgroundColor = .green
        homeVC.title = "Home"
        navigationController.pushViewController(homeVC, animated: true)
    }
}
```

---

### 4. Coordinator с делегатом для обратной связи

```swift
protocol LoginDelegate: AnyObject {
    func didLogin()
}

class LoginCoordinator: Coordinator, LoginDelegate {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        let loginVC = UIViewController()
        loginVC.view.backgroundColor = .white
        navigationController.pushViewController(loginVC, animated: true)
    }

    func didLogin() {
        let homeVC = UIViewController()
        homeVC.view.backgroundColor = .green
        navigationController.pushViewController(homeVC, animated: true)
    }
}
```

---

### 5. SwiftUI Coordinator (управление NavigationStack)

```swift
import SwiftUI

class AppCoordinator: ObservableObject {
    @Published var path = NavigationPath()

    func goToDetail() {
        path.append("Detail")
    }
}

struct ContentView: View {
    @StateObject var coordinator = AppCoordinator()

    var body: some View {
        NavigationStack(path: $coordinator.path) {
            VStack {
                Text("Home")
                Button("Go to Detail") {
                    coordinator.goToDetail()
                }
            }
            .navigationDestination(for: String.self) { value in
                Text("Screen: \(value)")
            }
        }
    }
}
```
