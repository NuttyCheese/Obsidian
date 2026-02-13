#tests #Swift 
Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по **iOSSnapshotTestCase** (FBSnapshotTestCase) в Swift — с современными практиками, альтернативами и рекомендациями.

### 1. Что такое iOSSnapshotTestCase и зачем он нужен в 2026 году

**iOSSnapshotTestCase** (часто называют просто **FBSnapshotTestCase**) — это популярный фреймворк для **снапшот-тестирования UI** в iOS-приложениях.

Суть работы:

1. Вы рендерите UIView / UIViewController / SwiftUI View в изображение  
2. Сравниваете это изображение с **эталонным** (записанным ранее)  
3. Если пиксели отличаются → тест падает → визуальная регрессия

**Главные преимущества в 2026 году**:

- Ловит **визуальные баги**, которые unit-тесты не видят (сдвиг пикселя, неправильный цвет, обрезка текста, неправильный шрифт)  
- Очень быстро пишутся и запускаются (обычно 0.1–2 секунды на тест)  
- Отлично работают с **CI/CD** (GitHub Actions, Bitrise, Xcode Cloud)  
- Позволяют **записывать эталоны** один раз и потом защищать их от регрессий  
- Поддерживают **разные размеры экрана**, **тёмную/светлую тему**, **локализацию**

**Актуальность 2026**:

- Для **UIKit** — **один из лучших** инструментов снапшот-тестирования  
- Для **SwiftUI** — частично вытеснен **SwiftUI snapshot testing** (Xcode 16+), но всё ещё очень популярен  
- Используется в **80–90%** крупных iOS-проектов с покрытием UI-тестами

### 2. Установка и настройка (2026 актуальные способы)

#### Способ 1 — Swift Package Manager (рекомендуемый)

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/uber/ios-snapshot-test-case.git", from: "8.0.0"),
]
```

#### Способ 2 — CocoaPods (всё ещё используется в legacy-проектах)

```ruby
pod 'iOSSnapshotTestCase', '~> 8.0'
```

После установки:

- В тест-таргете включить **Host Application** = ваш основной таргет  
- Добавить **Copy Bundle Resources** → все .xib / .storyboard / .xcassets

### 3. Самые популярные и рекомендуемые паттерны 2026 года

#### Паттерн 1 — Базовый тест UIViewController (UIKit)

```swift
import iOSSnapshotTestCase
@testable import MyApp

final class ProfileViewControllerTests: FBSnapshotTestCase {
    
    override func setUp() {
        super.setUp()
        recordMode = false  // true — для записи эталонов (один раз!)
        // fileNameOptions = [.screenSize, .device, .OS, .orientation] // по желанию
    }
    
    func testProfileViewControllerLightMode() {
        let vc = ProfileViewController()
        vc.loadViewIfNeeded()
        
        // Устанавливаем нужный trait (светлая тема, iPhone 15 Pro)
        vc.overrideUserInterfaceStyle = .light
        
        FBSnapshotVerifyViewController(vc)
        // или FBSnapshotVerifyView(vc.view, precision: 0.99)
    }
    
    func testProfileViewControllerDarkMode() {
        let vc = ProfileViewController()
        vc.loadViewIfNeeded()
        
        vc.overrideUserInterfaceStyle = .dark
        
        FBSnapshotVerifyViewController(vc)
    }
}
```

#### Паттерн 2 — Тест SwiftUI View (гибридный подход)

```swift
import SwiftUI
import iOSSnapshotTestCase

final class ProfileViewTests: FBSnapshotTestCase {
    
    func testProfileViewLight() {
        let view = ProfileView()
            .frame(width: 390, height: 844) // iPhone 14/15 размер
            .background(Color(.systemBackground))
        
        FBSnapshotVerifyView(UIViewWrapper(view: view), precision: 0.99)
    }
}

// Вспомогательный wrapper для SwiftUI → UIView
struct UIViewWrapper<V: View>: UIViewRepresentable {
    let view: V
    
    func makeUIView(context: Context) -> UIView {
        UIHostingController(rootView: view).view!
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {}
}
```

#### Паттерн 3 — Тест с разными состояниями (loading, error, empty)

```swift
func testLoadingState() {
    let vm = ProfileViewModel(state: .loading)
    let vc = ProfileViewController(viewModel: vm)
    vc.loadViewIfNeeded()
    
    FBSnapshotVerifyViewController(vc, identifier: "loading")
}

func testErrorState() {
    let vm = ProfileViewModel(state: .error(NSError(domain: "", code: -1)))
    let vc = ProfileViewController(viewModel: vm)
    vc.loadViewIfNeeded()
    
    FBSnapshotVerifyViewController(vc, identifier: "error")
}
```

**identifier** — ключевой параметр для разных состояний одного экрана.

### 4. Лучшие практики iOSSnapshotTestCase в 2026 году

- **recordMode = true** — запускайте **только один раз** для записи эталонов  
- **precision: 0.99–0.999** — 1.0 слишком строго (из-за антиалиасинга и рендеринга)  
- **identifier** — используйте для разных состояний одного VC/View  
- **Тёмная/светлая тема** — всегда тестируйте оба режима  
- **Разные размеры экранов** — добавляйте `.screenSize` в fileNameOptions  
- **Локализация** — тестируйте хотя бы en + ru (или вашу основную)  
- **SwiftUI** — используйте `UIViewWrapper` или **SwiftUI snapshot testing** (Xcode 16+)  
- **CI/CD** — записывайте эталоны локально → коммитьте в репозиторий → CI проверяет  
- **Git LFS** — обязательно для хранения .png эталонов  
- **Swift 6 strict concurrency** — тесты должны быть @MainActor или использовать `await MainActor.run`

### 5. Альтернативы iOSSnapshotTestCase в 2026 году

| Инструмент                     | Плюсы                                      | Минусы                                   | Рекомендация 2026 |
|--------------------------------|--------------------------------------------|------------------------------------------|-------------------|
| **iOSSnapshotTestCase**        | Зрелый, мощный, поддержка UIKit/SwiftUI   | Требует ручной записи эталонов           | ★★★★★ (UIKit + гибрид) |
| **SwiftUI snapshot testing**   | Встроено в Xcode 16+, @Observable         | Только SwiftUI, меньше гибкости          | ★★★★★ (чистый SwiftUI) |
| **PixelTest**                  | Очень быстрая запись/сравнение            | Меньше сообщество                        | ★★★★☆             |
| **SnapshotTesting (Point-Free)** | Очень гибкий, поддержка diff’ов           | Крутая кривая обучения                   | ★★★★☆ (TCA проекты) |
| **ViewInspector**              | Тестирование SwiftUI без снапшотов        | Не ловит визуальные баги                 | ★★★☆☆             |

### 6. Итог: когда и сколько снапшот-тестов писать в 2026

- **Пиши снапшоты** на все **экраны** и **ключевые состояния** (loading, content, error, empty, dark mode)  
- **Не пиши** на мелкие кастомные UIView без состояния  
- **Цель** — **70–90% покрытия UI-экранов** (а не 100% всех view)  
- **Лучший баланс** — быстрые, читаемые, устойчивые тесты, которые ловят визуальные регрессии

**Короткий девиз 2026**:
> «Снапшот-тесты — это страховка от визуальных багов и регрессий в UI.  
> В 2026 году это один из самых эффективных способов держать качество UI под контролем без огромных затрат времени.»

Удачи с надёжными, быстрыми и окупаемыми снапшот-тестами в Swift! 📸