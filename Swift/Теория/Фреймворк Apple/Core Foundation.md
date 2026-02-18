**Core Foundation** (CF) — это низкоуровневая библиотека на языке C, входящая в состав **Foundation** и являющаяся фундаментом почти всей экосистемы Apple ([[iOS]], iPadOS, macOS, watchOS, tvOS, visionOS).

По состоянию на февраль 2026 года Core Foundation остаётся **одним из самых важных и активно используемых** фреймворков, особенно в следующих случаях:

- взаимодействие с [[Objective-C]] [[Runtime]]  
- работа с низкоуровневыми типами данных (CFString, CFArray, CFDictionary и т.д.)  
- интеграция с C/C++ библиотеками  
- высокопроизводительные задачи, где [[Swift]]-объекты слишком «тяжёлые»  
- legacy-код и поддержка старых версий ОС

### Основные типы и возможности Core Foundation (актуальные 2026)

| Тип / Класс CF                     | Эквивалент в [[Swift]] / [[Foundation]] | Когда используют в 2026 году              | Примечание / статус |
| ---------------------------------- | --------------------------------------- | ----------------------------------------- | ------------------- |
| `CFStringRef`                      | [[String]] / [[NSString]]               | Почти никогда напрямую (кроме C-API)      | Toll-free bridged   |
| `CFArrayRef` / `CFMutableArrayRef` | `[Any]` / `NSMutableArray`              | Редко, только в C-интерфейсах             | Toll-free bridged   |
| `CFDictionaryRef`                  | `[AnyHashable: Any]` / `NSDictionary`   | Редко, в legacy или C-API                 | Toll-free bridged   |
| `CFDataRef`                        | [[Data]]                                | Часто в C-API (например, Security, Audio) | Toll-free bridged   |
| `CFURLRef`                         | [[URL]]                                 | Почти всегда `URL`                        | Toll-free bridged   |
| `CFUUIDRef`                        | [[UUID]]                                | Иногда в C-API                            | Toll-free bridged   |
| `CFBundleRef`                      | `Bundle`                                | Редко, в основном `Bundle.main`           | Toll-free bridged   |
| `CFErrorRef`                       | `Error` / `NSError`                     | Часто в C-API (Security, Network)         | Toll-free bridged   |
| `CFRunLoopRef`                     | `RunLoop`                               | Редко, в низкоуровневых задачах           | Не bridged          |
| `CFNotificationCenterRef`          | [[NotificationCenter]]                  | Почти всегда `NotificationCenter`         | Не bridged          |
| `CFMachPortRef`                    | —                                       | Очень редко (IPC, XPC)                    | Не bridged          |

### Самые частые сценарии использования Core Foundation в Swift 2026

1. **Работа с C-API библиотек** (Security, Audio, Core Graphics, Core Text, [[Core Location]] и т.д.)

```swift
import Security

func getCertificateData() -> Data? {
    var cert: SecCertificate?
    // ... получение сертификата через Security API
    
    guard let cert else { return nil }
    
    var data: CFData?
    let status = SecCertificateCopyData(cert, &data)
    
    guard status == errSecSuccess, let cfData = data else {
        return nil
    }
    
    return data as Data  // CFData → Data (toll-free)
}
```

2. **Toll-free bridging** (самый частый случай)

```swift
let cfString: CFString = "Hello" as CFString
let swiftString = cfString as String  // бесплатно и безопасно

let cfArray = [1, 2, 3] as CFArray
let swiftArray = cfArray as [Int]
```

3. **CFRunLoop** — низкоуровневое управление циклом событий (редко)

```swift
let runLoop = CFRunLoopGetCurrent()
CFRunLoopRunInMode(CFRunLoopMode.defaultMode.rawValue, 1.0, false)
```

4. **CFNotificationCenter** — иногда вместо NotificationCenter (legacy)

```swift
let center = CFNotificationCenterGetDarwinNotifyCenter()
CFNotificationCenterAddObserver(center, nil, { _, _, name, _, _ in
    print("Darwin notification: \(name ?? "nil")")
}, "com.example.myNotification" as CFString, nil, .deliverImmediately)
```

### Лучшие практики Core Foundation в Swift 2026

- **Используй Swift-типы** (`String`, `Array`, `Dictionary`, `Data`, `URL`, `UUID`, `Bundle`) вместо CF-типов, когда это возможно  
- **Toll-free bridging** — всегда безопасно и бесплатно (CFString → String, CFData → Data и т.д.)  
- **CF-типы** нужны только при работе с **C-API**, где Swift-объект не подходит  
- **Memory management** — CF-типы **не ARC**, используй `CFRelease` / `CFRetain` только при прямой работе с Core Foundation (редко)  
- **Swift 6 strict concurrency** — CF-типы **не Sendable** → передавай их только в `@MainActor` или через `unsafe` контекст  
- **Избегай** прямой работы с `CFRunLoop`, `CFMachPort` — это legacy  
- **Документируйте** — пиши комментарий «CFString → используется только для C-API Security»

**Короткий девиз 2026**:
> «Core Foundation в 2026 году — это когда тебе нужно общаться с низкоуровневыми C-API Apple.  
> В чистом Swift почти всё заменено на [[String]], [[Data]], [[Array]], [[Dictionary]] и т.д.  
> CF-типы используй только там, где без них не обойтись — и всегда через toll-free bridging.»
