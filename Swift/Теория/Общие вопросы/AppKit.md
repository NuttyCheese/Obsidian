**AppKit** — это основной фреймворк Apple для создания **нативных пользовательских интерфейсов** на **macOS**.

По состоянию на февраль 2026 года AppKit остаётся **единственным полноценным фреймворком** для разработки классических macOS-приложений (не Catalyst и не SwiftUI-only).

### Краткое сравнение AppKit с другими UI-фреймворками Apple (2026)

| Фреймворк            | Платформа                                      | Основное назначение                   | Статус в 2026 году              | Рекомендация для новых проектов                 |
| -------------------- | ---------------------------------------------- | ------------------------------------- | ------------------------------- | ----------------------------------------------- |
| **AppKit**           | macOS только                                   | Классические десктоп-приложения       | ★★★★★ (активно развивается)     | Основной для десктоп-приложений                 |
| **[[UIKit]]**        | iOS, iPadOS                                    | Мобильные и планшетные приложения     | ★★★★★                           | —                                               |
| **[[SwiftUI]]**      | iOS 13+, macOS 10.15+, watchOS, tvOS, visionOS | Декларативный UI для всех платформ    | ★★★★★ (главный фокус Apple)     | Рекомендуется для новых проектов                |
| **Catalyst**         | iPadOS → macOS                                 | iPad-приложения на Mac                | ★★★★☆ (стабильно, но не растёт) | Только для портирования iPad-приложений         |
| **AppKit + SwiftUI** | macOS                                          | Гибрид: AppKit-окна + SwiftUI-контент | ★★★★★ (самый популярный гибрид) | Идеально для большинства новых macOS-приложений |

### Ключевые изменения в AppKit к 2026 году (macOS 15+ / Sequoia и Sonoma)

- Полная поддержка **SwiftUI внутри AppKit** (NSHostingView / NSHostingController)  
- **NSWindowScene** и **Scene-based lifecycle** (с macOS 14 Sonoma)  
- Улучшенная поддержка **тёмной/светлой темы**, **Dynamic Type**, **аксессуаров**  
- **Metal-ускорение** для сложных view (NSView с Metal backing)  
- **Accessibility** — значительно улучшена интеграция с VoiceOver и Switch Control  
- **Swift 6 strict concurrency** — почти все AppKit-классы помечены как [[@MainActor]]  
- **[[SwiftUI]] + AppKit гибрид** стал де-факто стандартом для новых приложений

### Самые популярные и рекомендуемые паттерны AppKit в 2026 году

#### Паттерн 1 — Гибрид AppKit + SwiftUI (самый частый)

```swift
import AppKit
import SwiftUI

class MainWindowController: NSWindowController {
    
    convenience init() {
        let hosting = NSHostingController(rootView: ContentView())
        let window = NSWindow(
            contentRect: NSRect(x: 0, y: 0, width: 800, height: 600),
            styleMask: [.titled, .closable, .miniaturizable, .resizable],
            backing: .buffered,
            defer: false
        )
        window.center()
        window.contentViewController = hosting
        self.init(window: window)
    }
}

struct ContentView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") { count += 1 }
        }
        .padding()
        .frame(minWidth: 400, minHeight: 300)
    }
}
```

#### Паттерн 2 — Современный NSWindowDelegate + Scene-based lifecycle

```swift
class AppDelegate: NSObject, NSApplicationDelegate {
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        // Создаём первое окно через Scene
        let window = NSWindow(
            contentRect: NSRect(x: 0, y: 0, width: 1000, height: 700),
            styleMask: [.titled, .closable, .miniaturizable, .resizable, .fullSizeContentView],
            backing: .buffered,
            defer: false
        )
        window.titlebarAppearsTransparent = true
        window.titleVisibility = .hidden
        window.backgroundColor = .windowBackgroundColor
        
        window.contentView = NSHostingView(rootView: MainContentView())
        window.center()
        window.makeKeyAndOrderFront(nil)
    }
}
```

#### Паттерн 3 — Использование [[@MainActor]] и [[async]]/[[await]] в AppKit

```swift
@MainActor
class DocumentViewController: NSViewController {
    
    @Published var content: String = "Загрузка..."
    
    func loadDocument() async {
        do {
            let data = try await fetchDocumentData()
            content = String(decoding: data, as: UTF8.self)
        } catch {
            content = "Ошибка: \(error.localizedDescription)"
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            await loadDocument()
        }
    }
}
```

### Сравнение AppKit vs SwiftUI в 2026 году

| Критерий                                 | AppKit (2026)           | SwiftUI (2026)             | Победитель для новых проектов    |
| ---------------------------------------- | ----------------------- | -------------------------- | -------------------------------- |
| Скорость разработки                      | ★★★☆☆                   | ★★★★★                      | SwiftUI                          |
| Производительность сложных UI            | ★★★★★                   | ★★★★☆                      | AppKit (в очень тяжёлых случаях) |
| Поддержка macOS 10.13–14                 | ★★★★★                   | ★★☆☆☆                      | AppKit                           |
| Поддержка кастомных NSView               | ★★★★★                   | ★★☆☆☆                      | AppKit                           |
| Интеграция с [[Swift Concurrency]]       | ★★★★★ (@MainActor)      | ★★★★★                      | Ничья                            |
| Тестирование UI                          | [[XCUITest]] + Snapshot | SwiftUI Preview + Snapshot | SwiftUI                          |
| Общий рейтинг для новых macOS-приложений | ★★★★☆                   | ★★★★★                      | **SwiftUI + AppKit hybrid**      |

### Лучшие практики AppKit в 2026 году

- **Гибридный подход** — используй **SwiftUI внутри AppKit** (NSHostingView) для большинства экранов  
- **@MainActor** — пометь все ViewController и ViewModel  
- **async/await** — для всех сетевых, файловых и вычислительных операций  
- **Scene-based lifecycle** — используй `NSWindowScene` и `NSWindowDelegate`  
- **Accessibility** — всегда задавай `accessibilityIdentifier`, `accessibilityLabel`  
- **Тёмная/светлая тема** — используй `NSAppearance` и `effectiveAppearance`  
- **Swift 6 strict concurrency** — весь UI-код должен быть `@MainActor`  
- **Тестирование** — XCUITest + iOSSnapshotTestCase для визуальных регрессий

**Короткий девиз 2026**:
> «AppKit в 2026 году — это когда тебе нужен полный контроль над десктопным UI или поддержка старых macOS-версий.  
> Для большинства новых приложений — **SwiftUI + AppKit hybrid** (SwiftUI для контента, AppKit для окон/меню/тулбаров).  
> Чистый AppKit без SwiftUI — уже редкость.»
