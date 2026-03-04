#extension #uicolor #cgcolor

---
# UIKit — Расширения для UIColor

Коллекция удобных методов и инициализаторов для работы с цветами в UIKit.

```swift
// Генератор случайных цветов
extension UIColor {
    /// Случайный яркий цвет (RGB 0...1)
    static var random: UIColor {
        UIColor(
            red: .random(in: 0...1),
            green: .random(in: 0...1),
            blue: .random(in: 0...1),
            alpha: 1.0
        )
    }
    
    /// Случайный цвет из заданной палитры
    static func randomFromPalette(_ palette: [UIColor]) -> UIColor {
        palette.randomElement() ?? .systemBlue
    }
    
    /// Случайный пастельный / светлый цвет
    static func randomPastelColor() -> UIColor {
        UIColor(
            red:   (.random(in: 0...1) + 0.7) / 2,
            green: (.random(in: 0...1) + 0.7) / 2,
            blue:  (.random(in: 0...1) + 0.7) / 2,
            alpha: 1.0
        )
    }
}

// ────────────────────────────────────────────────
// Удобные инициализаторы

extension UIColor {
    /// UIColor из компонентов 0...255
    convenience init(red: UInt8, green: UInt8, blue: UInt8, alpha: UInt8 = 255) {
        self.init(
            red:   CGFloat(red)   / 255.0,
            green: CGFloat(green) / 255.0,
            blue:  CGFloat(blue)  / 255.0,
            alpha: CGFloat(alpha) / 255.0
        )
    }
    
    /// UIColor из HEX-числа (24-бит RGB или 32-бит ARGB)
    convenience init(hex: Int) {
        if hex > 0xFFFFFF {
            // ARGB
            self.init(
                red:   UInt8((hex >> 24) & 0xFF),
                green: UInt8((hex >> 16) & 0xFF),
                blue:  UInt8((hex >> 8)  & 0xFF),
                alpha: UInt8( hex        & 0xFF)
            )
        } else {
            // RGB
            self.init(
                red:   UInt8((hex >> 16) & 0xFF),
                green: UInt8((hex >> 8)  & 0xFF),
                blue:  UInt8( hex        & 0xFF)
            )
        }
    }
    
    /// UIColor из HEX-строки (#RRGGBB, RRGGBB, #AARRGGBB и т.д.)
    convenience init(hex: String) {
        var hexSanitized = hex.trimmingCharacters(in: .whitespacesAndNewlines).uppercased()
        hexSanitized = hexSanitized.replacingOccurrences(of: "#", with: "")
        
        let hexValue = UInt(hexSanitized, radix: 16) ?? 0
        
        self.init(hex: Int(hexValue))
    }
    
    /// Цвет из Asset Catalog (с fallback на systemBlue)
    static func appColor(named name: String) -> UIColor {
        if let color = UIColor(named: name) {
            return color
        }
        if let color = UIColor(named: "ColorSet/\(name)") {
            return color
        }
        return .systemBlue
    }
}

// ────────────────────────────────────────────────
// Динамические цвета (light / dark mode)

@available(iOS 13.0, *)
extension UIColor {
    /// Динамический цвет, меняющийся в зависимости от темы
    static func dynamicColor(light: UIColor, dark: UIColor) -> UIColor {
        UIColor { traitCollection in
            traitCollection.userInterfaceStyle == .dark ? dark : light
        }
    }
    
    /// Динамический цвет из HEX-строк
    static func dynamicHex(light: String, dark: String) -> UIColor {
        dynamicColor(
            light: UIColor(hex: light),
            dark:  UIColor(hex: dark)
        )
    }
}

// ────────────────────────────────────────────────
// Градиент (самый частый запрос)

extension UIColor {
    /// **НЕВОЗМОЖНО** создать UIColor, который сам по себе является градиентом  
    /// UIColor — это всегда один однородный цвет.  
    ///  
    /// Вместо этого почти всегда создают **CAGradientLayer**  
    /// Вот самый удобный и часто копируемый хелпер:
    static func gradientLayer(
        colors: [UIColor],
        frame: CGRect,
        startPoint: CGPoint = CGPoint(x: 0.0, y: 0.5),
        endPoint: CGPoint   = CGPoint(x: 1.0, y: 0.5)
    ) -> CAGradientLayer {
        let layer = CAGradientLayer()
        layer.frame = frame
        layer.colors = colors.map { $0.cgColor }
        layer.startPoint = startPoint
        layer.endPoint   = endPoint
        return layer
    }
    
    /// Удобный шорткат для горизонтального градиента
    static func horizontalGradientLayer(
        colors: UIColor...,
        frame: CGRect
    ) -> CAGradientLayer {
        gradientLayer(
            colors: Array(colors),
            frame: frame,
            startPoint: CGPoint(x: 0, y: 0.5),
            endPoint: CGPoint(x: 1, y: 0.5)
        )
    }
    
    /// Удобный шорткат для вертикального градиента
    static func verticalGradientLayer(
        colors: UIColor...,
        frame: CGRect
    ) -> CAGradientLayer {
        gradientLayer(
            colors: Array(colors),
            frame: frame,
            startPoint: CGPoint(x: 0.5, y: 0),
            endPoint: CGPoint(x: 0.5, y: 1)
        )
    }
}
```

### Типичные примеры использования градиента

```swift
// Вариант 1 — добавляем слой в view
let gradient = UIColor.horizontalGradientLayer(
    colors: .systemPurple, .systemPink, .systemOrange,
    frame: view.bounds
)

view.layer.insertSublayer(gradient, at: 0)

// Вариант 2 — в кастомной UIView
override func layoutSubviews() {
    super.layoutSubviews()
    
    let gradientLayer = UIColor.gradientLayer(
        colors: [.systemIndigo, .systemTeal],
        frame: bounds,
        startPoint: .zero,
        endPoint: CGPoint(x: 1, y: 1)  // диагональ
    )
    
    layer.insertSublayer(gradientLayer, at: 0)
}

// Вариант 3 — для кнопки / карточки
button.layer.insertSublayer(
    UIColor.gradientLayer(colors: [.systemBlue, .systemCyan], frame: button.bounds),
    at: 0
)
```

