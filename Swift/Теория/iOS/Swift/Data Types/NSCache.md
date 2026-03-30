**`NSCache`** — это **потокобезопасный кэш** из Foundation, предназначенный для временного хранения объектов в памяти с автоматическим удалением при нехватке RAM.

В отличие от обычного [[Dictionary]], `NSCache`:
- сам чистит старые/редко используемые объекты,
- не удерживает сильные ссылки на ключи,
- полностью потокобезопасен (можно читать/писать из любого потока без lock'ов).

Это делает его **идеальным** для кэширования изображений, аватарок, результатов сетевых запросов, thumbnail'ов, parsed [[JSON]] и любых других дорогих объектов, которые можно пересоздать.

### 1. Ключевые отличия NSCache от Dictionary (2025–2026)

| Характеристика                              | NSCache                                 | Dictionary                     | Когда выбирать NSCache         |
| ------------------------------------------- | --------------------------------------- | ------------------------------ | ------------------------------ |
| Потокобезопасность                          | Да (встроенная)                         | Нет (нужен lock или [[actor]]) | Всегда, если многопоточность   |
| Автоматическое удаление при memory pressure | Да (очень агрессивно)                   | Нет                            | Основная причина использовать  |
| Ограничение по количеству объектов          | `countLimit`                            | Нет                            | Да, если нужен лимит           |
| Ограничение по «стоимости» (размер)         | `totalCostLimit` + `cost` при setObject | Нет                            | Идеально для изображений       |
| Сильные ссылки на ключи                     | Нет (ключи weak)                        | Да                             | Нет [[retain cycle]]s на ключи |
| Сильные ссылки на значения                  | Да (пока объект в кэше)                 | Да                             | —                              |
| Производительность вставки/чтения           | Очень высокая                           | Высокая                        | Ничья                          |
| Подходит для больших объёмов                | Нет (только кэш)                        | Да (если хватает памяти)       | NSCache — только кэш           |

### 2. Самый популярный и правильный паттерн 2026 (image cache)

```swift
final class ImageCache {
    static let shared = ImageCache()
    
    private let cache: NSCache<NSString, UIImage> = {
        let cache = NSCache<NSString, UIImage>()
        cache.name = "ImageCache"
        cache.countLimit = 150               // макс. 150 изображений
        cache.totalCostLimit = 80 * 1024 * 1024 // ≈80 МБ
        cache.evictsObjectsWithDiscardedContent = true
        return cache
    }()
    
    private init() {
        // Подписываемся на memory warning (очень рекомендуется)
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(clearCache),
            name: UIApplication.didReceiveMemoryWarningNotification,
            object: nil
        )
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
    
    func set(_ image: UIImage, for url: URL) {
        cache.setObject(image, forKey: url.absoluteString as NSString, cost: image.cost)
    }
    
    func get(for url: URL) -> UIImage? {
        cache.object(forKey: url.absoluteString as NSString)
    }
    
    func remove(for url: URL) {
        cache.removeObject(forKey: url.absoluteString as NSString)
    }
    
    @objc private func clearCache() {
        cache.removeAllObjects()
    }
}

// Расширение для UIImage — оценка стоимости в байтах
extension UIImage {
    var cost: Int {
        guard let cgImage else { return 0 }
        return cgImage.width * cgImage.height * cgImage.bitsPerPixel / 8
    }
}
```

### 3. Как правильно использовать NSCache в реальном коде

#### 3.1 Кэширование изображений из сети (самый частый кейс)

```swift
func loadImage(from url: URL, completion: @escaping (UIImage?) -> Void) {
    // 1. Проверяем кэш
    if let cached = ImageCache.shared.get(for: url) {
        completion(cached)
        return
    }
    
    // 2. Загружаем
    URLSession.shared.dataTask(with: url) { data, _, error in
        guard let data, error == nil, let image = UIImage(data: data) else {
            completion(nil)
            return
        }
        
        // 3. Сохраняем в кэш
        ImageCache.shared.set(image, for: url)
        completion(image)
    }.resume()
}
```

#### 3.2 Кэширование результатов API (JSON → Decodable)

```swift
final class APICache {
    static let shared = APICache()
    
    private let cache = NSCache<NSString, NSData>()
    
    func cache<T: Decodable>(_ object: T, for url: URL) throws {
        let data = try JSONEncoder().encode(object)
        cache.setObject(data as NSData, forKey: url.absoluteString as NSString)
    }
    
    func object<T: Decodable>(for url: URL, type: T.Type) throws -> T? {
        guard let data = cache.object(forKey: url.absoluteString as NSString) as? Data else {
            return nil
        }
        return try JSONDecoder().decode(T.self, from: data)
    }
}
```

### 4. Лучшие практики NSCache в Swift 2026

- **Всегда** задавай `countLimit` и/или `totalCostLimit` — иначе кэш будет расти бесконтрольно  
- **Используй `cost`** для изображений (размер в байтах) — это самый точный лимит  
- **Подписывайся на `UIApplication.didReceiveMemoryWarningNotification`** и чисти кэш  
- **Используй `weak` ссылки** на ключи и значения, если это возможно (но NSCache уже делает ключи [[weak]])  
- **Не храни в NSCache** ничего, что нельзя быстро пересоздать (пароли, токены, критичные данные)  
- **Swift 6 strict concurrency** — NSCache полностью потокобезопасен, но сам объект кэша лучше держать в [[@MainActor]] или отдельном акторе  
- **Документируйте** — пиши комментарий «NSCache — кэш аватарок, лимит 150 изображений / 80 МБ»

**Короткий девиз 2026**:
> `NSCache` — это «умный словарь», который сам выкидывает старые записи, когда памяти мало.  
> В 2026 году используй его **везде**, где нужно кэшировать:  
> - изображения / аватарки  
> - результаты [[API]] / [[JSON]]  
> - thumbnail'ы  
> - parsed данные  
> Главное — задавай `countLimit` и `totalCostLimit` + чисти при memory warning.
