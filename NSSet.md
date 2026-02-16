Типичные места, где NSSet до сих пор появляется:

- `userInfo` в `NSNotification`  
- `value(forKey:)` в [[KVC]]  
- `NSSet` из [[Core Data]] (to-many relationship без order)  
- `-[NSObject valueForKeyPath:]` и другие старые [[API]]  
- JSONSerialization (если где-то остался старый код)

### Как выглядит NSSet в отладчике / print

```text
NSSet: {(
    "tag1",
    "tag3",
    42,
    <User: 0x12345678>
)}
```

Или в логе:

```
<NSSet: 0x600000123456> {(
    "value1",
    "value2"
)}
```

### Как правильно работать с NSSet в Swift 2026

Самый надёжный и современный подход — **сразу приводить к Swift [[Set]]**.

```swift
// Вариант 1 — самый частый и безопасный
if let nsSet = someObject.value(forKey: "tags") as? NSSet {
    let swiftSet = Set(nsSet.compactMap { $0 as? String })
    // или
    let tags: Set<String> = Set(nsSet.compactMap { $0 as? String })
}

// Вариант 2 — если уверен в типе
let tags = nsSet as? Set<String> ?? []

// Вариант 3 — если объекты AnyHashable
let anySet: Set<AnyHashable> = nsSet as Set<AnyHashable>
```

### Краткая шпаргалка «NSSet → Swift Set»

| Что пришло                  | Как привести                                                           | Результат          |
| --------------------------- | ---------------------------------------------------------------------- | ------------------ |
| `NSSet` с [[String]]        | `nsSet as? Set<String>` или compactMap                                 | `Set<String>`      |
| `NSSet` с разными типами    | `nsSet as Set<AnyHashable>`                                            | `Set<AnyHashable>` |
| `NSSet` из Core Data        | `let set = relationship as? Set<MyEntity>`                             | `Set<MyEntity>`    |
| `userInfo["key"] as? NSSet` | `Set((userInfo["key"] as? NSSet)?.compactMap { $0 as? String } ?? [])` | `Set<String>`      |

### Короткий вывод 2026

> NSSet — это **Objective-C артефакт**, который иногда вылезает из-под старых API.  
> Как только увидел NSSet — сразу делай `Set<...>` и забудь про него.  
> В чистом Swift 2026 года NSSet не нужен почти никогда.
