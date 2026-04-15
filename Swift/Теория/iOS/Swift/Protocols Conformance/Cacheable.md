#caching #performance #swift #protocol #memory #disk #ios

---
## Cacheable — протокол для кэширования данных в Swift
### Определение

**`Cacheable`** — это не встроенный протокол в [[Swift]], а **соглашение (концепция)**, которое вы можете реализовать в своих типах данных. Он означает, что данные могут быть сохранены в кэш и восстановлены из него. Обычно `Cacheable` требует, чтобы тип поддерживал:

- Преобразование в [[Data]] для сохранения
- Инициализацию из `Data` для восстановления
- Уникальный идентификатор для ключа

Фактически, `Cacheable` — это **[[Codable]] + [[Identifiable]]** для кэширования.

---

### Зачем это знать iOS-разработчику?

1.  **Производительность:** Кэширование данных (изображений, ответов [[API]], результатов вычислений) ускоряет приложение и экономит трафик.
2.  **Офлайн-режим:** Кэш позволяет приложению работать без интернета.
3.  **Снижение нагрузки на сервер:** Меньше запросов → дешевле и быстрее.
4.  **Архитектура:** Паттерн Repository использует кэш как один из источников данных.
5.  **Пользовательский опыт:** Быстрая загрузка повторяющегося контента.

---

### Основные протоколы для Cacheable в Swift

В стандартной библиотеке нет единого протокола `Cacheable`, но есть несколько строительных блоков:

| Протокол             | Назначение                           |
| -------------------- | ------------------------------------ |
| **[[Codable]]**      | Преобразование в `Data` и обратно    |
| **[[Identifiable]]** | Уникальный идентификатор для ключа   |
| **[[Hashable]]**     | Использование объекта как ключа кэша |

#### Типичная реализация Cacheable:

```swift
protocol Cacheable: Codable, Identifiable where ID == String {
    var id: ID { get }
    var cacheKey: String { get }
}

extension Cacheable {
    var cacheKey: String {
        return "\(Self.self)-\(id)"
    }
}
```

---

### Простые примеры

#### 1. **Модель, поддерживающая кэширование**

```swift
import Foundation

struct User: Cacheable {
    let id: String
    let name: String
    let email: String
    let avatarURL: URL?
}

// Автоматически:
// - Codable синтезирован
// - id строка
// - cacheKey = "User-123"
```

#### 2. **Сохранение в память ([[NSCache]])**

```swift
class MemoryCache<Key: Hashable, Value: Cacheable> {
    private let cache = NSCache<NSString, NSData>()
    
    func set(_ value: Value) {
        guard let data = try? JSONEncoder().encode(value) else { return }
        cache.setObject(data as NSData, forKey: value.cacheKey as NSString)
    }
    
    func get(for key: String) -> Value? {
        guard let data = cache.object(forKey: key as NSString) as? Data else { return nil }
        return try? JSONDecoder().decode(Value.self, from: data)
    }
}

// Использование
let userCache = MemoryCache<String, User>()
let user = User(id: "123", name: "Alice", email: "alice@example.com", avatarURL: nil)
userCache.set(user)

if let cached = userCache.get(for: "User-123") {
    print(cached.name)  // "Alice"
}
```

#### 3. **Сохранение на диск ([[FileManager]])**

```swift
class DiskCache<Value: Cacheable> {
    private let cacheDirectory: URL
    
    init(cacheName: String) {
        let paths = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask)
        cacheDirectory = paths[0].appendingPathComponent(cacheName, isDirectory: true)
        try? FileManager.default.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)
    }
    
    func set(_ value: Value) {
        let fileURL = cacheDirectory.appendingPathComponent(value.cacheKey).appendingPathExtension("json")
        guard let data = try? JSONEncoder().encode(value) else { return }
        try? data.write(to: fileURL)
    }
    
    func get(for key: String) -> Value? {
        let fileURL = cacheDirectory.appendingPathComponent(key).appendingPathExtension("json")
        guard let data = try? Data(contentsOf: fileURL) else { return nil }
        return try? JSONDecoder().decode(Value.self, from: data)
    }
    
    func clear() {
        try? FileManager.default.removeItem(at: cacheDirectory)
    }
}
```

---

### Продвинутый пример: Двухуровневый кэш (память + диск)

```swift
class TieredCache<Value: Cacheable> {
    private let memoryCache = NSCache<NSString, NSData>()
    private let diskCache: DiskCache<Value>
    
    init(cacheName: String) {
        diskCache = DiskCache<Value>(cacheName: cacheName)
    }
    
    func set(_ value: Value) {
        // Сохраняем в память
        if let data = try? JSONEncoder().encode(value) {
            memoryCache.setObject(data as NSData, forKey: value.cacheKey as NSString)
        }
        // Сохраняем на диск
        diskCache.set(value)
    }
    
    func get(for key: String) -> Value? {
        // Сначала проверяем память
        if let data = memoryCache.object(forKey: key as NSString) as? Data,
           let value = try? JSONDecoder().decode(Value.self, from: data) {
            return value
        }
        
        // Если нет — проверяем диск
        if let value = diskCache.get(for: key) {
            // Восстанавливаем в память для следующего раза
            set(value)
            return value
        }
        
        return nil
    }
}
```

---

### Cacheable и Repository Pattern

```swift
protocol UserRepositoryProtocol {
    func getUser(id: String) async throws -> User
}

class UserRepository: UserRepositoryProtocol {
    private let apiService: APIService
    private let cache: TieredCache<User>
    
    init(apiService: APIService, cache: TieredCache<User>) {
        self.apiService = apiService
        self.cache = cache
    }
    
    func getUser(id: String) async throws -> User {
        let cacheKey = "User-\(id)"
        
        // Сначала проверяем кэш
        if let cached = cache.get(for: cacheKey) {
            return cached
        }
        
        // Если нет — запрашиваем с сервера
        let user = try await apiService.fetchUser(id: id)
        cache.set(user)
        return user
    }
}
```

---

### Cacheable для изображений ([[UIImage]])

Изображения не поддерживают `Codable` по умолчанию, но их можно адаптировать:

```swift
struct CachedImage: Cacheable {
    let id: String
    let imageData: Data
    
    var uiImage: UIImage? {
        return UIImage(data: imageData)
    }
    
    init(id: String, image: UIImage) {
        self.id = id
        self.imageData = image.pngData() ?? Data()
    }
    
    init(id: String, data: Data) {
        self.id = id
        self.imageData = data
    }
}

// Использование
let imageCache = TieredCache<CachedImage>(cacheName: "Images")
let avatar = CachedImage(id: "user-123-avatar", image: userAvatarImage)
imageCache.set(avatar)
```

---

### Cacheable и NSCache (оптимизация памяти)

`NSCache` автоматически удаляет объекты при нехватке памяти:

```swift
class AutoClearingMemoryCache<Key: Hashable, Value: Cacheable> {
    private let cache = NSCache<NSString, NSData>()
    
    init(countLimit: Int = 50, totalCostLimit: Int = 5 * 1024 * 1024) {
        cache.countLimit = countLimit
        cache.totalCostLimit = totalCostLimit  // 5 MB
    }
    
    func set(_ value: Value) {
        guard let data = try? JSONEncoder().encode(value) else { return }
        cache.setObject(data as NSData, forKey: value.cacheKey as NSString)
    }
    
    func get(for key: String) -> Value? {
        guard let data = cache.object(forKey: key as NSString) as? Data else { return nil }
        return try? JSONDecoder().decode(Value.self, from: data)
    }
}
```

---

### Когда использовать Cacheable

| Сценарий                      | Использовать Cacheable | Почему                          |
| ----------------------------- | ---------------------- | ------------------------------- |
| **Ответы API**                | ✅                      | Снижение нагрузки на сервер     |
| **Изображения**               | ✅                      | Экономия трафика и ускорение UI |
| **Результаты вычислений**     | ✅                      | Ускорение повторных операций    |
| **Конфигурации приложения**   | ✅                      | Быстрый доступ                  |
| **Временные данные (сессия)** | ❌                      | Используйте память напрямую     |
| **Секретные данные (токены)** | ❌                      | Используйте [[Keychain]]        |

---

### Лучшие практики

1.  **Всегда задавай лимиты для кэша** — без лимитов память может переполниться.
2.  **Используй двухуровневый кэш (память + диск)** — баланс скорости и персистентности.
3.  **Очищай кэш при выходе из приложения (опционально)** — для чувствительных данных.
4.  **Используй `NSCache` для памяти** — он автоматически реагирует на предупреждения системы.
5.  **Добавляй версионирование кэша** — при обновлении формата данных старый кэш должен быть сброшен.

```swift
struct CacheConfig {
    static let version = 2
    static let cacheKeyPrefix = "v\(version)-"
}
```

---

### Итог

**`Cacheable`** — это концепция, а не встроенный протокол, но она критически важна для производительности iOS-приложений.

| Тип кэша | Скорость | Персистентность | Объём |
|---|---|---|---|
| **Память (NSCache)** | ★★★★★ | ❌ (при убийстве приложения) | Маленький |
| **Диск (FileManager)** | ★★★☆☆ | ✅ | Большой |
| **Двухуровневый** | ★★★★☆ | ✅ | Большой |

**Ключевые выводы:**
- Используйте `Codable` + `Identifiable` для реализации `Cacheable`
- `NSCache` — для памяти, `FileManager` — для диска
- Двухуровневый кэш даёт лучший баланс скорости и персистентности
- Всегда задавайте лимиты, чтобы избежать переполнения памяти