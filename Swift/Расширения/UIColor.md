```swift
@available(iOS 13.0, *)
extension UIColor {
    /// Создает динамический цвет, который автоматически меняется между светлой и темной темой
    static func dynamicColor(light: UIColor, dark: UIColor) -> UIColor {
        return UIColor { traitCollection in
            return traitCollection.userInterfaceStyle == .dark ? dark : light
        }
    }
    
    /// HEX для динамических цветов
    static func dynamicHex(light: String, dark: String) -> UIColor {
        return dynamicColor(
            light: UIColor(hex: light),
            dark: UIColor(hex: dark)
        )
    }
}

// 2. Улучшенный инициализатор с валидацией
extension UIColor {
    convenience init(hex: String, defaultColor: UIColor = .systemBlue) {
        var hexSanitized = hex.trimmingCharacters(in: .whitespacesAndNewlines)
        hexSanitized = hexSanitized.replacingOccurrences(of: "#", with: "")
        
        // Валидация длины
        guard hexSanitized.count == 6 || hexSanitized.count == 8 else {
            self.init(cgColor: defaultColor.cgColor)
            return
        }
        
        // Валидация символов
        let isValid = hexSanitized.allSatisfy { char in
            char.isHexDigit
        }
        
        guard isValid else {
            self.init(cgColor: defaultColor.cgColor)
            return
        }
        
        let hex = UInt(hexSanitized, radix: 16) ?? 0
        self.init(hex: Int(hex))
    }
}

// 3. Методы для работы с градиентами
extension UIColor {
    /// Создает градиентный слой из массива цветов
    static func gradientLayer(colors: [UIColor], 
                             startPoint: CGPoint = CGPoint(x: 0, y: 0),
                             endPoint: CGPoint = CGPoint(x: 1, y: 0)) -> CAGradientLayer {
        let gradient = CAGradientLayer()
        gradient.colors = colors.map { $0.cgColor }
        gradient.startPoint = startPoint
        gradient.endPoint = endPoint
        return gradient
    }
    
    /// Создает градиентное изображение
    static func gradientImage(size: CGSize, 
                             colors: [UIColor],
                             locations: [NSNumber]? = nil) -> UIImage? {
        let gradientLayer = gradientLayer(colors: colors)
        gradientLayer.frame = CGRect(origin: .zero, size: size)
        gradientLayer.locations = locations
        
        UIGraphicsBeginImageContext(size)
        guard let context = UIGraphicsGetCurrentContext() else { return nil }
        
        gradientLayer.render(in: context)
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        
        return image
    }
}

// 4. Цветовые манипуляции
extension UIColor {
    /// Осветляет цвет на указанный процент
    func lighten(by percentage: CGFloat) -> UIColor {
        return adjust(by: abs(percentage))
    }
    
    /// Затемняет цвет на указанный процент
    func darken(by percentage: CGFloat) -> UIColor {
        return adjust(by: -abs(percentage))
    }
    
    private func adjust(by percentage: CGFloat) -> UIColor {
        var red: CGFloat = 0
        var green: CGFloat = 0
        var blue: CGFloat = 0
        var alpha: CGFloat = 0
        
        getRed(&red, green: &green, blue: &blue, alpha: &alpha)
        
        return UIColor(
            red: min(red + percentage/100, 1.0),
            green: min(green + percentage/100, 1.0),
            blue: min(blue + percentage/100, 1.0),
            alpha: alpha
        )
    }
    
    /// Возвращает цвет с измененной прозрачностью
    func withAlpha(_ alpha: CGFloat) -> UIColor {
        return self.withAlphaComponent(alpha)
    }
    
    /// Инвертирует цвет
    var inverted: UIColor {
        var red: CGFloat = 0
        var green: CGFloat = 0
        var blue: CGFloat = 0
        var alpha: CGFloat = 0
        
        getRed(&red, green: &green, blue: &blue, alpha: &alpha)
        
        return UIColor(
            red: 1.0 - red,
            green: 1.0 - green,
            blue: 1.0 - blue,
            alpha: alpha
        )
    }
}

// 5. Расширение для CGColor
extension CGColor {
    static func fromHex(_ hex: String) -> CGColor {
        return UIColor(hex: hex).cgColor
    }
}

// 6. Улучшенный appColor с кэшированием
extension UIColor {
    private static var colorCache: [String: UIColor] = [:]
    
    static func cachedAppColor(named name: String) -> UIColor {
        let cacheKey = "color_\(name)"
        
        if let cachedColor = colorCache[cacheKey] {
            return cachedColor
        }
        
        let color = appColor(named: name)
        colorCache[cacheKey] = color
        
        return color
    }
    
    static func clearColorCache() {
        colorCache.removeAll()
    }
}

// 7. Поддержка цветовых пространств
extension UIColor {
    /// Конвертирует цвет в sRGB цветовое пространство
    var sRGB: UIColor? {
        return self.cgColor.converted(
            to: CGColorSpace(name: CGColorSpace.sRGB)!,
            intent: .defaultIntent,
            options: nil
        ).map { UIColor(cgColor: $0) }
    }
    
    /// Конвертирует цвет в P3 цветовое пространство (широкая гамма)
    var displayP3: UIColor? {
        return self.cgColor.converted(
            to: CGColorSpace(name: CGColorSpace.displayP3)!,
            intent: .defaultIntent,
            options: nil
        ).map { UIColor(cgColor: $0) }
    }
}

// 8. Генератор случайных цветов
extension UIColor {
    static var random: UIColor {
        return UIColor(
            red: .random(in: 0...1),
            green: .random(in: 0...1),
            blue: .random(in: 0...1),
            alpha: 1.0
        )
    }
    
    static func randomFromPalette(_ palette: [UIColor]) -> UIColor {
        return palette.randomElement() ?? .systemBlue
    }
    
    static func randomPastelColor() -> UIColor {
        return UIColor(
            red: (.random(in: 0...1) + 0.7) / 2,
            green: (.random(in: 0...1) + 0.7) / 2,
            blue: (.random(in: 0...1) + 0.7) / 2,
            alpha: 1.0
        )
    }
}

// 9. Цветовые константы для часто используемых цветов
extension UIColor {
    static let facebookBlue = UIColor(hex: "#1877F2")
    static let twitterBlue = UIColor(hex: "#1DA1F2")
    static let instagramPurple = UIColor(hex: "#E1306C")
    static let linkedinBlue = UIColor(hex: "#0077B5")
    static let youtubeRed = UIColor(hex: "#FF0000")
    static let githubBlack = UIColor(hex: "#181717")
    static let googleRed = UIColor(hex: "#DB4437")
    static let appleBlack = UIColor(hex: "#000000")
    
    static let paypalBlue = UIColor(hex: "#003087")
    static let stripePurple = UIColor(hex: "#6772E5")
    static let visaBlue = UIColor(hex: "#1A1F71")
    static let mastercardRed = UIColor(hex: "#EB001B")
    
    static let bitcoinOrange = UIColor(hex: "#F7931A")
    static let ethereumPurple = UIColor(hex: "#627EEA")
}
```