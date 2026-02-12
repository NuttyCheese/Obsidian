```swift
extension Bool {
    var intValue: Int { self ? 1 : 0 }
    var int8Value: Int8 { self ? 1 : 0 }
    var int16Value: Int16 { self ? 1 : 0 }
    var int32Value: Int32 { self ? 1 : 0 }
    var int64Value: Int64 { self ? 1 : 0 }
    var floatValue: Float { self ? 1.0 : 0.0 }
    var doubleValue: Double { self ? 1.0 : 0.0 }
    
    // Для NSNumber (Objective-C совместимость)
    var numberValue: NSNumber { NSNumber(value: self.intValue) }
}

// 2. Обратное преобразование
extension Int {
    var boolValue: Bool { self != 0 }
}

// 3. Локализованные строки
extension Bool {
    var localizedYesNo: String {
        self ? NSLocalizedString("yes", comment: "") 
             : NSLocalizedString("no", comment: "")
    }
    
    var localizedEnabled: String {
        self ? NSLocalizedString("enabled", comment: "") 
             : NSLocalizedString("disabled", comment: "")
    }
}

// 4. Операции с массивами булевых значений
extension Array where Element == Bool {
    var trueCount: Int { self.filter { $0 }.count }
    var falseCount: Int { self.filter { !$0 }.count }
    var allTrue: Bool { !self.contains(false) }
    var allFalse: Bool { !self.contains(true) }
    
    func toIntArray() -> [Int] {
        self.map { $0.toInt() }
    }
}
```