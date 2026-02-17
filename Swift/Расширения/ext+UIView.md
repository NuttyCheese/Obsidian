#extension #uiview 
```swift
extension UIView {
    /// Включает пользовательское взаимодействие
    func enableUserInteraction() {
        isUserInteractionEnabled = true
    }
    
    /// Выключает пользовательское взаимодействие
    func disableUserInteraction() {
        isUserInteractionEnabled = false
    }
    
    /// Отключает autoresizing mask для Auto Layout
    func disableAutoresizingMask() {
        translatesAutoresizingMaskIntoConstraints = false
    }
    
    /// Добавляет несколько subviews
    func addSubviews(_ views: UIView...) {
        views.forEach { addSubview($0) }
    }
    
    /// Добавляет несколько subviews из массива
    func addSubviews(_ views: [UIView]) {
        views.forEach { addSubview($0) }
    }
}

// 2. Методы для работы с углами
extension UIView {
    /// Закругляет указанные углы
    func roundCorners(_ corners: UIRectCorner, radius: CGFloat) {
        let path = UIBezierPath(
            roundedRect: bounds,
            byRoundingCorners: corners,
            cornerRadii: CGSize(width: radius, height: radius)
        )
        
        let mask = CAShapeLayer()
        mask.path = path.cgPath
        layer.mask = mask
    }
    
    /// Закругляет все углы
    func roundAllCorners(radius: CGFloat) {
        layer.cornerRadius = radius
        layer.masksToBounds = true
    }
    
    /// Добавляет границу
    func addBorder(color: UIColor, width: CGFloat) {
        layer.borderColor = color.cgColor
        layer.borderWidth = width
    }
}

// 3. Методы для теней
extension UIView {
    /// Добавляет тень
    func addShadow(color: UIColor = .black,
                  opacity: Float = 0.1,
                  offset: CGSize = .zero,
                  radius: CGFloat = 4) {
        layer.shadowColor = color.cgColor
        layer.shadowOpacity = opacity
        layer.shadowOffset = offset
        layer.shadowRadius = radius
        layer.masksToBounds = false
    }
    
    /// Убирает тень
    func removeShadow() {
        layer.shadowColor = nil
        layer.shadowOpacity = 0
    }
    
    /// Добавляет внутреннюю тень
    func addInnerShadow(color: UIColor = .black,
                       opacity: Float = 0.1,
                       radius: CGFloat = 4) {
        let shadowLayer = CAShapeLayer()
        shadowLayer.frame = bounds
        
        shadowLayer.shadowColor = color.cgColor
        shadowLayer.shadowOffset = CGSize(width: 0, height: 0)
        shadowLayer.shadowOpacity = opacity
        shadowLayer.shadowRadius = radius
        shadowLayer.fillRule = .evenOdd
        
        let path = UIBezierPath(rect: bounds)
        path.append(UIBezierPath(rect: bounds.insetBy(dx: -radius, dy: -radius)))
        shadowLayer.path = path.cgPath
        
        layer.addSublayer(shadowLayer)
    }
}

// 4. Методы для анимаций
extension UIView {
    /// Простая анимация fade in
    func fadeIn(duration: TimeInterval = 0.3, 
               completion: (() -> Void)? = nil) {
        alpha = 0
        isHidden = false
        
        UIView.animate(withDuration: duration, animations: {
            self.alpha = 1
        }) { _ in
            completion?()
        }
    }
    
    /// Простая анимация fade out
    func fadeOut(duration: TimeInterval = 0.3,
                completion: (() -> Void)? = nil) {
        UIView.animate(withDuration: duration, animations: {
            self.alpha = 0
        }) { _ in
            self.isHidden = true
            completion?()
        }
    }
    
    /// Анимация нажатия
    func animateTap(scale: CGFloat = 0.95, 
                   completion: (() -> Void)? = nil) {
        UIView.animate(withDuration: 0.1, animations: {
            self.transform = CGAffineTransform(scaleX: scale, y: scale)
        }) { _ in
            UIView.animate(withDuration: 0.1, animations: {
                self.transform = .identity
            }) { _ in
                completion?()
            }
        }
    }
    
    /// Анимация встряхивания (для ошибок)
    func shake() {
        let animation = CAKeyframeAnimation(keyPath: "transform.translation.x")
        animation.timingFunction = CAMediaTimingFunction(name: .linear)
        animation.duration = 0.6
        animation.values = [-20, 20, -20, 20, -10, 10, -5, 5, 0]
        layer.add(animation, forKey: "shake")
    }
}

// 5. Методы для работы с констрейнтами
extension UIView {
    /// Привязывает вью ко всем сторонам другой вью
    func pinToSuperview(insets: UIEdgeInsets = .zero) {
        guard let superview = superview else { return }
        
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            topAnchor.constraint(equalTo: superview.topAnchor, constant: insets.top),
            leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: insets.left),
            trailingAnchor.constraint(equalTo: superview.trailingAnchor, constant: -insets.right),
            bottomAnchor.constraint(equalTo: superview.bottomAnchor, constant: -insets.bottom)
        ])
    }
    
    /// Привязывает вью к safe area
    func pinToSafeArea(insets: UIEdgeInsets = .zero) {
        guard let superview = superview else { return }
        
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            topAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.topAnchor, constant: insets.top),
            leadingAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.leadingAnchor, constant: insets.left),
            trailingAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.trailingAnchor, constant: -insets.right),
            bottomAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.bottomAnchor, constant: -insets.bottom)
        ])
    }
    
    /// Центрирует вью внутри другой вью
    func centerInSuperview() {
        guard let superview = superview else { return }
        
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            centerXAnchor.constraint(equalTo: superview.centerXAnchor),
            centerYAnchor.constraint(equalTo: superview.centerYAnchor)
        ])
    }
    
    /// Устанавливает размер вью
    func setSize(width: CGFloat? = nil, height: CGFloat? = nil) {
        disableAutoresizingMask()
        
        var constraints: [NSLayoutConstraint] = []
        
        if let width = width {
            constraints.append(widthAnchor.constraint(equalToConstant: width))
        }
        
        if let height = height {
            constraints.append(heightAnchor.constraint(equalToConstant: height))
        }
        
        NSLayoutConstraint.activate(constraints)
    }
}

// 6. Методы для работы с иерархией вью
extension UIView {
    /// Удаляет все subviews
    func removeAllSubviews() {
        subviews.forEach { $0.removeFromSuperview() }
    }
    
    /// Находит родительский ViewController
    var parentViewController: UIViewController? {
        var parentResponder: UIResponder? = self
        while parentResponder != nil {
            parentResponder = parentResponder?.next
            if let viewController = parentResponder as? UIViewController {
                return viewController
            }
        }
        return nil
    }
    
    /// Находит все subviews определенного типа
    func subviews<T: UIView>(ofType type: T.Type) -> [T] {
        var result: [T] = []
        
        for subview in subviews {
            if let typedSubview = subview as? T {
                result.append(typedSubview)
            }
            result.append(contentsOf: subview.subviews(ofType: type))
        }
        
        return result
    }
}

// 7. Методы для создания скриншотов
extension UIView {
    /// Создает скриншот вью
    func takeScreenshot() -> UIImage? {
        UIGraphicsBeginImageContextWithOptions(bounds.size, false, UIScreen.main.scale)
        
        defer { UIGraphicsEndImageContext() }
        
        guard let context = UIGraphicsGetCurrentContext() else { return nil }
        
        layer.render(in: context)
        return UIGraphicsGetImageFromCurrentImageContext()
    }
    
    /// Создает скриншот определенной области
    func takeScreenshot(of rect: CGRect) -> UIImage? {
        UIGraphicsBeginImageContextWithOptions(rect.size, false, UIScreen.main.scale)
        
        defer { UIGraphicsEndImageContext() }
        
        guard let context = UIGraphicsGetCurrentContext() else { return nil }
        
        context.translateBy(x: -rect.origin.x, y: -rect.origin.y)
        layer.render(in: context)
        
        return UIGraphicsGetImageFromCurrentImageContext()
    }
}

// 8. Методы для градиентов
extension UIView {
    /// Добавляет градиентный слой
    func addGradient(colors: [UIColor],
                    startPoint: CGPoint = CGPoint(x: 0, y: 0),
                    endPoint: CGPoint = CGPoint(x: 1, y: 0),
                    locations: [NSNumber]? = nil) {
        let gradient = CAGradientLayer()
        gradient.frame = bounds
        gradient.colors = colors.map { $0.cgColor }
        gradient.startPoint = startPoint
        gradient.endPoint = endPoint
        gradient.locations = locations
        
        layer.insertSublayer(gradient, at: 0)
    }
    
    /// Обновляет градиент при изменении размера
    func updateGradientFrame() {
        layer.sublayers?
            .compactMap { $0 as? CAGradientLayer }
            .forEach { $0.frame = bounds }
    }
}

// 9. Методы для вибрации (haptic feedback)
extension UIView {
    /// Легкая вибрация
    func lightHapticFeedback() {
        let generator = UIImpactFeedbackGenerator(style: .light)
        generator.impactOccurred()
    }
    
    /// Средняя вибрация
    func mediumHapticFeedback() {
        let generator = UIImpactFeedbackGenerator(style: .medium)
        generator.impactOccurred()
    }
    
    /// Сильная вибрация
    func heavyHapticFeedback() {
        let generator = UIImpactFeedbackGenerator(style: .heavy)
        generator.impactOccurred()
    }
    
    /// Вибрация успеха
    func successHapticFeedback() {
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
    }
    
    /// Вибрация ошибки
    func errorHapticFeedback() {
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.error)
    }
}

// 10. Утилиты для debug
extension UIView {
    /// Показывает границы вью и всех subviews (для debug)
    func showDebugBorders(color: UIColor = .red) {
        layer.borderColor = color.cgColor
        layer.borderWidth = 1
        
        subviews.forEach { $0.showDebugBorders(color: color) }
    }
    
    /// Убирает debug границы
    func hideDebugBorders() {
        layer.borderWidth = 0
        subviews.forEach { $0.hideDebugBorders() }
    }
    
    /// Выводит иерархию вью в консоль
    func printViewHierarchy(level: Int = 0) {
        let indent = String(repeating: "  ", count: level)
        print("\(indent)\(String(describing: type(of: self))) - frame: \(frame)")
        
        for subview in subviews {
            subview.printViewHierarchy(level: level + 1)
        }
    }
}
```