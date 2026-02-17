#collection #string #optional
```swift
extension Collection {
    /// Проверяет, содержит ли коллекция хотя бы один элемент,
    /// удовлетворяющий условию
    func hasAny(where predicate: (Element) -> Bool) -> Bool {
        contains(where: predicate)
    }
    
    /// Проверяет, что все элементы удовлетворяют условию
    func allSatisfy(_ predicate: (Element) -> Bool) -> Bool {
        allSatisfy(predicate)
    }
    
    /// Проверяет, что ни один элемент не удовлетворяет условию
    func hasNone(where predicate: (Element) -> Bool) -> Bool {
        !contains(where: predicate)
    }
}

// 2. Безопасные геттеры
extension Collection {
    /// Безопасно возвращает первый элемент, если коллекция не пуста
    var firstIfNotEmpty: Element? {
        isNotEmpty ? first : nil
    }
    
    /// Безопасно возвращает последний элемент, если коллекция не пуста
    var lastIfNotEmpty: Element? {
        isNotEmpty ? last : nil
    }
    
    /// Возвращает элемент по безопасному индексу
    func get(at index: Index) -> Element? {
        indices.contains(index) ? self[index] : nil
    }
}

// 3. Для строк
extension String {
    var isNotEmpty: Bool { !isEmpty }
    
    /// Проверяет, не является ли строка пустой или состоящей только из пробелов
    var isNotBlank: Bool {
        !trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }
}

// 4. Для Optional коллекций
extension Optional where Wrapped: Collection {
    var isNilOrEmpty: Bool {
        self?.isEmpty ?? true
    }
    
    var isNotEmpty: Bool {
        !isNilOrEmpty
    }
}

// 5. Агрегация для коллекций булевых значений
extension Collection where Element == Bool {
    /// Количество true значений
    var trueCount: Int {
        filter { $0 }.count
    }
    
    /// Количество false значений
    var falseCount: Int {
        filter { !$0 }.count
    }
    
    /// Все ли значения true
    var allTrue: Bool {
        !contains(false)
    }
    
    /// Все ли значения false
    var allFalse: Bool {
        !contains(true)
    }
}
```