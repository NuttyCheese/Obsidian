#network #Swift 
**URLCache** — это встроенный класс в Foundation, который отвечает за **кэширование ответов** на сетевые запросы ([[URLRequest]] → [[URLResponse]] + [[Data]]).

Он позволяет:
- автоматически сохранять ответы сервера
- быстро возвращать данные из кэша без повторного запроса
- экономить трафик, ускорять приложение
- работать оффлайн (частично)

### 1. Когда и зачем использовать URLCache

| Сценарий                                  | Стоит ли использовать URLCache? | Почему                                 |
| ----------------------------------------- | ------------------------------- | -------------------------------------- |
| Загрузка изображений, аватарок, иконок    | Да (очень часто)                | Повторные загрузки из кэша — мгновенно |
| Статические списки (города, категории)    | Да                              | Данные редко меняются                  |
| [[API]] с ETag / Cache-Control            | Да                              | Автоматическая валидация кэша          |
| Часто повторяющиеся запросы (профиль)     | Да                              | Снижение нагрузки на сервер            |
| Динамические данные (новости, лента)      | Иногда                          | Только с правильной политикой кэша     |
| Критически важные данные (токены, личные) | Нет                             | Риск устаревших данных                 |

### 2. Основные свойства и методы URLCache

| Свойство / Метод              | Тип                  | Что делает / возвращает                   | Когда использовать |
| ----------------------------- | -------------------- | ----------------------------------------- | ------------------ |
| `shared`                      | `URLCache`           | Глобальный кэш по умолчанию               | Быстрый старт      |
| `memoryCapacity`              | [[Int]]              | Максимальный объём в памяти (в байтах)    | Настройка размера  |
| `diskCapacity`                | `Int`                | Максимальный объём на диске               | Настройка размера  |
| `currentMemoryUsage`          | `Int`                | Текущий объём в памяти                    | Отладка            |
| `currentDiskUsage`            | `Int`                | Текущий объём на диске                    | Отладка            |
| `cachedResponse(for:)`        | `CachedURLResponse?` | Возвращает кэшированный ответ для запроса | Ручной доступ      |
| `storeCachedResponse(_:for:)` | `Void`               | Сохраняет ответ в кэш вручную             | Кастомный кэш      |
| `removeCachedResponse(for:)`  | `Void`               | Удаляет конкретный ответ                  | Очистка            |
| `removeAllCachedResponses()`  | `Void`               | Полная очистка кэша                       | Логаут, очистка    |

### 3. Создание и настройка URLCache

#### Вариант 1 — Использование глобального shared (самый частый)

```swift
// По умолчанию: 20–50 МБ памяти + 100–200 МБ диска
let cache = URLCache.shared

// Или кастомная настройка (рекомендуется для продакшена)
let memoryCapacity = 25 * 1024 * 1024   // 25 МБ
let diskCapacity   = 150 * 1024 * 1024  // 150 МБ
let cache = URLCache(memoryCapacity: memoryCapacity,
                     diskCapacity: diskCapacity,
                     diskPath: "myAppCache")

// Установка как глобальный кэш
URLCache.shared = cache
```

#### Вариант 2 — Кастомная конфигурация для [[URLSession]]

```swift
let config = URLSessionConfiguration.default
config.urlCache = URLCache(memoryCapacity: 50 * 1024 * 1024,
                           diskCapacity: 200 * 1024 * 1024,
                           diskPath: "networkCache")
config.requestCachePolicy = .returnCacheDataElseLoad

let customSession = URLSession(configuration: config)
```

### 4. Политики кэширования (requestCachePolicy)

| Политика                              | Когда возвращает кэш                  | Когда делает запрос | Когда обновляет кэш | Лучше всего для |
|---------------------------------------|----------------------------------------|----------------------|---------------------|-----------------|
| `.useProtocolCachePolicy` (default)   | Если сервер разрешил (Cache-Control)   | Если кэш устарел    | Да                  | Большинство случаев |
| `.returnCacheDataElseLoad`            | Всегда, если есть кэш                  | Только если нет     | Нет                 | Оффлайн-режим   |
| `.reloadIgnoringLocalCacheData`       | Никогда                                | Всегда              | Да                  | Свежие данные   |
| `.reloadIgnoringLocalAndRemoteCacheData` | Никогда                             | Всегда              | Да                  | Критические данные |
| `.returnCacheDataDontLoad`            | Только если есть кэш                   | Никогда             | Нет                 | Только оффлайн  |

### 5. Реальные примеры использования (2026)

#### Пример 1 — Автоматическое кэширование изображений

```swift
let config = URLSessionConfiguration.default
config.urlCache = URLCache(memoryCapacity: 100 * 1024 * 1024,
                           diskCapacity: 500 * 1024 * 1024,
                           diskPath: "imagesCache")
config.requestCachePolicy = .returnCacheDataElseLoad

let imageSession = URLSession(configuration: config)

func loadImage(from url: URL) async throws -> UIImage {
    let request = URLRequest(url: url)
    let (data, _) = try await imageSession.data(for: request)
    guard let image = UIImage(data: data) else {
        throw URLError(.cannotDecodeContentData)
    }
    return image
}
```

#### Пример 2 — Ручное сохранение и получение ответа

```swift
func cacheResponseManually(url: URL, data: Data, mimeType: String) {
    let response = HTTPURLResponse(url: url,
                                  statusCode: 200,
                                  httpVersion: "1.1",
                                  headerFields: ["Content-Type": mimeType])!
    
    let cached = CachedURLResponse(response: response, data: data)
    URLCache.shared.storeCachedResponse(cached, for: URLRequest(url: url))
}

func getCachedData(for url: URL) -> Data? {
    URLCache.shared.cachedResponse(for: URLRequest(url: url))?.data
}
```

#### Пример 3 — Очистка кэша по условиям

```swift
func clearCacheForDomain(_ domain: String) {
    let cache = URLCache.shared
    guard let allKeys = cache.allCachedResponses().map({ $0.request.url?.host }) else { return }
    
    for response in cache.allCachedResponses() {
        if response.request.url?.host == domain {
            cache.removeCachedResponse(for: response.request)
        }
    }
}

func clearAllCache() {
    URLCache.shared.removeAllCachedResponses()
}
```

#### Пример 4 — Кэширование с учётом авторизации

```swift
func fetchProfile(token: String) async throws -> Profile {
    guard let url = URL(string: "https://api.example.com/profile") else { throw URLError(.badURL) }
    
    var request = URLRequest(url: url)
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    
    // Важно: разные токены — разные кэшированные ответы
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(Profile.self, from: data)
}
```

### 6. Таблица: политики кэша и сценарии

| Политика                              | Когда кэш возвращается                  | Когда запрос уходит на сервер | Лучше всего для |
|---------------------------------------|-----------------------------------------|-------------------------------|-----------------|
| `.useProtocolCachePolicy` (default)   | Если сервер разрешил (Cache-Control)    | Если кэш устарел              | Почти всегда    |
| `.returnCacheDataElseLoad`            | Если есть кэш                           | Только если нет кэша          | Оффлайн-режим   |
| `.reloadIgnoringLocalCacheData`       | Никогда                                 | Всегда                        | Свежие данные   |
| `.reloadIgnoringLocalAndRemoteCacheData` | Никогда                              | Всегда                        | Критические данные |
| `.returnCacheDataDontLoad`            | Только если есть кэш                    | Никогда                       | Только оффлайн  |

### 7. Типичные ошибки и как их избежать

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Нет настройки размера кэша                  | Быстро заполняется диск                      | Установить 50–200 МБ diskCapacity |
| Кэширование авторизованных данных без токена| Показ чужих данных                           | Добавить токен в заголовки |
| Игнорирование Cache-Control от сервера      | Устаревшие данные                            | Использовать `.useProtocolCachePolicy` |
| Не очищать кэш при логауте                  | Утечка приватных данных                      | Вызывать `removeAllCachedResponses()` |
| Использование shared без настройки          | Слишком маленький кэш по умолчанию           | Создать свой URLCache |

### 8. Лучшие практики 2026 года

- Для изображений/статичных ресурсов — **отдельный URLCache** с большим diskCapacity
- Для API с авторизацией — **не кэшировать** или добавлять токен в заголовки
- Для оффлайн-режима — `.returnCacheDataElseLoad`
- Для свежих данных — `.reloadIgnoringLocalCacheData`
- Для отладки — логировать `currentDiskUsage` и `currentMemoryUsage`
- Использовать **async/await** + `URLSession` с кастомным кэшем
- Очищать кэш при логауте / смене аккаунта

**Короткий девиз**:
> «URLCache — это бесплатное ускорение и оффлайн-режим.  
> Настраивай размер, политику и очищай при необходимости — и приложение будет летать.»
