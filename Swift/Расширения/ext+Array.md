#extension #collection #array
```swift
extension Collection {
    subscript(safe index: Index) -> Element? {
        indices.contains(index) ? self[index] : nil
    }
}

// 2. Безопасный доступ для диапазонов
extension Array {
    subscript(safe range: Range<Int>) -> ArraySlice<Element>? {
        guard range.lowerBound >= 0, 
              range.upperBound <= count else { return nil }
        return self[range]
    }
    
    subscript(safe range: ClosedRange<Int>) -> ArraySlice<Element>? {
        guard range.lowerBound >= 0, 
              range.upperBound < count else { return nil }
        return self[range]
    }
}

// 3. Thread-safe версия для многопоточных сред
class ThreadSafeArray<T> {
    private var array: [T] = []
    private let queue = DispatchQueue(
        label: "com.threadsafe.array", 
        attributes: .concurrent
    )
    
    subscript(safe index: Int) -> T? {
        queue.sync {
            indices.contains(index) ? array[index] : nil
        }
    }
}

// 4. Операции с опциональными результатами
extension Array {
    /// Безопасное применение трансформации
    func safeMap<T>(_ transform: (Element) -> T?) -> [T] {
        self.compactMap { element in
            transform(element)
        }
    }
    
    /// Безопасное получение и трансформация
    func get<T>(at index: Int, transform: (Element) -> T?) -> T? {
        self[safe: index].flatMap(transform)
    }
}

```