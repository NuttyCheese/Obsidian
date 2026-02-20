**UIColor** — это основной класс в **UIKit** для работы с цветами в iOS-приложениях (и в macOS через AppKit как NSColor).

Он представляет цвет в различных цветовых пространствах (sRGB, Display P3, grayscale, CMYK и др.), поддерживает альфа-канал, динамические цвета (адаптация под светлую/тёмную тему), цвета из ассетов и системные цвета.

В 2026 году UIColor остаётся центральным типом для всех UI-элементов UIKit, а в SwiftUI используется через `Color(uiColor:)`.

### Основные способы создания UIColor (самые используемые в 2026)

| Способ создания                              | Пример кода                                                                 | Когда использовать в 2026 |
|----------------------------------------------|-----------------------------------------------------------------------------|----------------------------|
| Системные цвета (самый частый и рекомендуемый) | `UIColor.systemBlue`, `UIColor.label`, `UIColor.systemBackground`           | Почти всегда — адаптируются к тёмной/светлой теме |
| RGB (0–1)                                    | `UIColor(red: 0.2, green: 0.4, blue: 0.8, alpha: 1.0)`                      | Когда нужен точный цвет из дизайна |
| RGB (0–255)                                  | `UIColor(red: 51/255, green: 102/255, blue: 204/255, alpha: 1)`             | Конвертация из Figma/Sketch |
| HEX (самый удобный для дизайнеров)           | `UIColor(hex: "#3366CC")` или `UIColor(hex: 0x3366CC)`                      | Импорт из дизайна (расширение) |
| HSB / HSV                                    | `UIColor(hue: 0.6, saturation: 0.8, brightness: 0.9, alpha: 1)`            | Генерация оттенков, градиентов |
| Из ассетов (Assets.xcassets)                 | `UIColor(named: "PrimaryColor")`                                            | Цвета из проекта (поддержка тёмной темы) |
| Dynamic (адаптация под тему)                 | `UIColor { trait in trait.userInterfaceStyle == .dark ? .black : .white }`  | Цвета, меняющиеся при смене темы |
| Grayscale                                    | `UIColor(white: 0.5, alpha: 1.0)`                                           | Монохромные элементы |

### Расширения для удобства (очень популярные в 2026)

```swift
extension UIColor {
    convenience init(hex: String, alpha: CGFloat = 1.0) {
        var hexString = hex.trimmingCharacters(in: .whitespacesAndNewlines).uppercased()
        if hexString.hasPrefix("#") { hexString.removeFirst() }
        
        var rgb: UInt64 = 0
        Scanner(string: hexString).scanHexInt64(&rgb)
        
        let r = CGFloat((rgb >> 16) & 0xFF) / 255.0
        let g = CGFloat((rgb >> 8)  & 0xFF) / 255.0
        let b = CGFloat( rgb        & 0xFF) / 255.0
        
        self.init(red: r, green: g, blue: b, alpha: alpha)
    }
    
    convenience init(hex: Int, alpha: CGFloat = 1.0) {
        let r = CGFloat((hex >> 16) & 0xFF) / 255.0
        let g = CGFloat((hex >> 8)  & 0xFF) / 255.0
        let b = CGFloat( hex        & 0xFF) / 255.0
        self.init(red: r, green: g, blue: b, alpha: alpha)
    }
    
    var hexString: String {
        var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0, a: CGFloat = 0
        getRed(&r, green: &g, blue: &b, alpha: &a)
        return String(format: "#%02lX%02lX%02lX", lroundf(Float(r*255)), lroundf(Float(g*255)), lroundf(Float(b*255)))
    }
}
```

### Dynamic цвета (адаптация под тёмную/светлую тему)

```swift
// Самый современный способ (iOS 13+)
let dynamicColor = UIColor { traitCollection in
    switch traitCollection.userInterfaceStyle {
    case .dark:
        return .systemBackground  // или кастомный тёмный
    case .light, .unspecified:
        return UIColor(red: 0.95, green: 0.95, blue: 0.97, alpha: 1)
    @unknown default:
        return .white
    }
}

// Использование
view.backgroundColor = dynamicColor
```

### Системные цвета (рекомендуется использовать в 99% случаев)

```swift
// Семантические цвета (адаптируются к теме)
UIColor.label              // основной текст
UIColor.secondaryLabel     // вторичный текст
UIColor.systemBackground   // фон приложения
UIColor.systemGroupedBackground // фон таблиц/групп
UIColor.systemBlue         // акцентный синий
UIColor.systemRed          // ошибки, удаление
UIColor.systemGreen        // успех, подтверждение
UIColor.systemGray         // разделители, неактивные элементы
```

### Лучшие практики UIColor в Swift 2026

- **Всегда** предпочитайте **системные цвета** (`systemBlue`, `label`, `systemBackground`) — они автоматически адаптируются к тёмной теме и accessibility  
- **Для кастомных цветов** — используйте **Assets.xcassets** → Color Set с поддержкой Light/Dark  
- **Для динамических цветов** — используйте конструктор `UIColor { trait in ... }`  
- **Для HEX** — добавьте расширение (как выше) — это ускоряет работу с дизайном  
- **В SwiftUI** — предпочитайте `Color` и `Color(uiColor:)`, но для UIKit → UIColor  
- **Для градиентов** — используйте `CAGradientLayer` с массивом `UIColor` → `.cgColor`  
- **Для доступности** — проверяйте контраст с помощью `UIColor.contrastRatio(between:)` (расширение)  
- **Документируйте** — пишите комментарий «UIColor — динамический акцентный цвет, адаптируется под тёмную/светлую тему»

**Короткий итог 2026**:
> UIColor — это **центральный класс** для работы с цветами в UIKit.  
> В 2026 году:  
> - 90% случаев — **системные цвета** (`systemBlue`, `label`, `systemBackground`)  
> - кастомные цвета — из **Assets.xcassets** с поддержкой Light/Dark  
> - динамические цвета — через `UIColor { trait in ... }`  
> - HEX и RGB — только для быстрого прототипирования  
> Это **самый безопасный** и **самый адаптивный** способ работы с цветом в iOS-приложениях.

Удачи с красивым, доступным и тематически правильным UI в твоём проекте! 🎨