## 1. Что такое NSCache

**NSCache** — это **потокобезопасная коллекция для хранения временных объектов**.

- Похожа на словарь (`Dictionary<Key, Value>`), но имеет особенности:
    
    - Автоматически **удаляет объекты при нехватке памяти**.
        
    - **Потокобезопасна** — можно использовать из разных потоков без ручной синхронизации.
        
    - Подходит для **кеша изображений, данных из сети, временных объектов**.
        

> Проще: NSCache = «умный словарь», который сам чистит память, когда нужно.

---

## 2. Основные свойства и методы

```swift
let cache = NSCache<NSString, UIImage>()

// Добавить объект
cache.setObject(myImage, forKey: "avatar")

// Получить объект
let image = cache.object(forKey: "avatar")

// Удалить объект
cache.removeObject(forKey: "avatar")

// Очистить весь кеш
cache.removeAllObjects()
```

- `countLimit` — ограничение на количество объектов
    
- `totalCostLimit` — ограничение на «стоимость» (например, размер в байтах)
    

```swift
cache.countLimit = 100
cache.totalCostLimit = 10 * 1024 * 1024 // 10 МБ
```

---

## 3. Потокобезопасность

- Можно читать и писать **из разных потоков**:
    

```swift
DispatchQueue.global().async {
    cache.setObject(image1, forKey: "1")
}
DispatchQueue.global().async {
    let img = cache.object(forKey: "1")
}
```

- Нет необходимости использовать [[NSLock]] или [[DispatchQueue]] для синхронизации.
    

---

## 4. Пример: кеширование изображений

```swift
let imageCache = NSCache<NSString, UIImage>()

func loadImage(from url: URL, completion: @escaping (UIImage?) -> Void) {
    if let cachedImage = imageCache.object(forKey: url.absoluteString as NSString) {
        completion(cachedImage) // вернули кеш
        return
    }
    
    URLSession.shared.dataTask(with: url) { data, _, _ in
        guard let data = data, let image = UIImage(data: data) else {
            completion(nil)
            return
        }
        imageCache.setObject(image, forKey: url.absoluteString as NSString)
        completion(image)
    }.resume()
}
```

- Если изображение уже загружено → сразу берём из кеша.
    
- Если нет → загружаем, сохраняем в кеш и возвращаем.
    

---

## 5. Отличие NSCache от [[Dictionary]]

|Свойство|NSCache|Dictionary|
|---|---|---|
|Потокобезопасность|✅|❌ (нужна синхронизация)|
|Автоматическое удаление|✅ при нехватке памяти|❌|
|Ограничение по количеству/размеру|✅|❌|
|Использование|Кеширование временных данных|Общие данные|

---

## 6. Итог

- **NSCache** — идеальный инструмент для кеширования в [[iOS]]/macOS.
    
- Основные преимущества:
    
    - потокобезопасность,
        
    - автоматическое удаление при нехватке памяти,
        
    - возможность задать ограничения.
        
- Используется для **изображений, данных из сети, временных объектов**.
    

---
