#crash #warning #xcode #swift
### Что это значит

[[Xcode]] предупреждает, что вы используете **устаревший [[API]]**, который был помечен как **deprecated** начиная с указанной версии [[iOS]].

- Deprecated API **ещё работает**, но **не рекомендуется к использованию**.
    
- Apple может удалить этот API в будущих версиях iOS → использование старого метода рискованно.
    
- Обычно в предупреждении указана **минимальная версия iOS**, с которой метод стал deprecated.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Использование deprecated метода**

```swift
let fileManager = FileManager.default
let urls = fileManager.urls(for: .documentDirectory, in: .userDomainMask) // ⚠️ 'urls(for:in:)' is deprecated: first deprecated in iOS 16.0
```

- Метод помечен как устаревший → Xcode предупреждает о возможной проблеме в будущем.
    

---

**Пример 2: Deprecated свойство**

```swift
let window = UIApplication.shared.keyWindow // ⚠️ 'keyWindow' is deprecated: first deprecated in iOS 13.0
```

- `keyWindow` устарел с iOS 13 → рекомендуется использовать `windows` и `first { $0.isKeyWindow }`.
    

---

### Как исправить

#### 1️⃣ Использовать новый рекомендованный API

```swift
// Вместо deprecated keyWindow
if let keyWindow = UIApplication.shared.connectedScenes
    .compactMap({ $0 as? UIWindowScene })
    .flatMap({ $0.windows })
    .first(where: { $0.isKeyWindow }) {
    print("Key window: \(keyWindow)")
}
```

```swift
// Вместо deprecated FileManager.urls(for:in:)
let urls = fileManager.directoryURLs(for: .documentDirectory, in: .userDomainMask)
```

---

#### 2️⃣ Проверять документацию Apple

- Каждый deprecated API сопровождается **заменой или рекомендацией**, как правильно делать сейчас.
    
- В Xcode можно нажать **Option + Click** на метод → увидите сообщение о deprecation.
    

---

#### 3️⃣ Обновление минимальной версии iOS (если возможно)

- Иногда API deprecated только в последних версиях iOS.
    
- Если вы поддерживаете старые версии, можно использовать `#available`:
    

```swift
if #available(iOS 16, *) {
    // использовать новый API
} else {
    // fallback для старых версий
}
```

---

### Резюме

- Предупреждение указывает на использование **устаревшего API**.
    
- Исправляется заменой на современный API, проверкой документации и использованием условий `#available`.
    
- Помогает поддерживать совместимость и предотвращает проблемы при будущих обновлениях iOS.
    
