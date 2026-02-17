**Хэш-поиск** — это метод поиска элемента по ключу с использованием **хэш-таблицы**. В Swift это основной способ достижения **амортизированной сложности O(1)** для операций поиска, вставки и удаления.

### Основные свойства

- Средняя сложность: **O(1)**
- Худший случай (много коллизий): **O(n)**
- Не требует сортировки данных
- Требует **хорошей хэш-функции** и уникальных ключей
- Встроенная реализация: `Dictionary<Key, Value>` и `Set<Element>` (где `Element: Hashable`)

### Когда хэш-поиск — лучший выбор в iOS (2026)

- Быстрый доступ по ID, username, token, [[UUID]]
- Кэширование (изображения, ответы [[API]], результаты запросов)
- Проверка уникальности ([[Set]])
- Группировка данных ([[Dictionary]] группировки)
- Фильтрация по ключу в больших коллекциях

## 1. Основной и самый частый способ — Dictionary

```swift
// Создание словаря
let users: [String: String] = [
    "alice123": "Alice Johnson",
    "bob_dev":  "Bob Smith",
    "john_ios": "John Doe"
]

// Поиск — O(1) в среднем
if let name = users["bob_dev"] {
    print("Найден: \(name)")           // Bob Smith
}

// Проверка существования
if users["alice123"] != nil {
    print("Пользователь alice123 существует")
}

// Безопасный доступ с дефолтным значением
let displayName = users["unknown"] ?? "Гость"
```

### Получение всех ключей / значений

```swift
let allUsernames = users.keys.sorted()           // ["alice123", "bob_dev", "john_ios"]
let allNames     = users.values.sorted()         // ["Alice Johnson", "Bob Smith", "John Doe"]
```

## 2. Создание словаря из массива структур (очень частый паттерн)

```swift
struct Product {
    let id: String
    let name: String
    let price: Double
    let category: String
}

let products = [
    Product(id: "P001", name: "iPhone 16", price: 999, category: "Phone"),
    Product(id: "P002", name: "MacBook Pro", price: 2499, category: "Laptop"),
    Product(id: "P003", name: "AirPods Pro", price: 249, category: "Accessories")
]

// Словарь для быстрого поиска по ID
let productsById = Dictionary(uniqueKeysWithValues: products.map { ($0.id, $0) })

// Поиск продукта по ID
if let product = productsById["P002"] {
    print("Найден: \(product.name) за $\(product.price)")
    // → MacBook Pro за $2499
}
```

## 3. Группировка через Dictionary (очень мощный паттерн)

```swift
// Группировка продуктов по категории
let productsByCategory = Dictionary(grouping: products) { $0.category }

// Пример вывода
for (category, items) in productsByCategory {
    print("\(category): \(items.map { $0.name }.joined(separator: ", "))")
}
// Phone: iPhone 16
// Laptop: MacBook Pro
// Accessories: AirPods Pro
```

## 4. Set — хэш-поиск для проверки уникальности / наличия

```swift
let activeUserIDs: Set<Int> = [1001, 1005, 1012, 1020, 1050]

// Проверка наличия — O(1)
if activeUserIDs.contains(1005) {
    print("Пользователь 1005 онлайн")
}

// Добавление с автоматической проверкой уникальности
var favorites = Set<String>()
favorites.insert("SwiftUI")
favorites.insert("Combine")
favorites.insert("SwiftUI")  // не добавится повторно

print(favorites)  // {"Combine", "SwiftUI"}
```

## 5. Реальные сценарии из iOS-приложений

### Сценарий 1: Кэш изображений по [[URL]]

```swift
var imageCache: [URL: UIImage] = [:]

// Загрузка или получение из кэша
func loadImage(for url: URL) async -> UIImage? {
    if let cached = imageCache[url] {
        return cached
    }
    
    // ... загрузка из сети
    if let image = await downloadImage(from: url) {
        imageCache[url] = image
        return image
    }
    
    return nil
}
```

### Сценарий 2: Быстрая проверка прав доступа

```swift
let userPermissions: Set<String> = ["read:profile", "write:posts", "delete:account"]

func canPerform(_ action: String) -> Bool {
    userPermissions.contains(action)
}

print(canPerform("write:posts"))      // true
print(canPerform("ban:user"))         // false
```

### Сценарий 3: Поиск пользователя по email (из API-ответа)

```swift
struct UserResponse: Codable {
    let id: Int
    let email: String
    let name: String
}

let apiUsers: [UserResponse] = // результат от сервера

// Создаём словарь для O(1) поиска
let usersByEmail = Dictionary(uniqueKeysWithValues: apiUsers.map { ($0.email, $0) })

// Быстрый поиск
if let user = usersByEmail["anna@example.com"] {
    print("Найден пользователь: \(user.name), ID: \(user.id)")
}
```

### Сценарий 4: Уникальные теги в постах

```swift
let postTags = ["swift", "ios", "swiftui", "combine", "swift", "ios"]

let uniqueTags = Set(postTags)
// → {"swift", "ios", "swiftui", "combine"}
```

## 6. Сравнение производительности

| Количество элементов | Dictionary / Set (среднее) | Линейный поиск ([[firstIndex]]) | Бинарный поиск |
| -------------------- | -------------------------- | ------------------------------- | -------------- |
| 100                  | ~0.000001 мс               | ~0.0001 мс                      | ~0.00001 мс    |
| 10 000               | ~0.00001 мс                | ~0.01 мс                        | ~0.0003 мс     |
| 1 000 000            | ~0.0001 мс                 | ~10 мс                          | ~0.001 мс      |

**Вывод**:  
Для поиска по ключу **Dictionary / Set** — это **золотой стандарт** в Swift.  
`firstIndex(where:)` используйте только если ключ не уникален или данные не подходят для хэширования.

## Итог — современные рекомендации (2026)

| Задача                                      | Что использовать                     |
|---------------------------------------------|--------------------------------------|
| Быстрый поиск по уникальному ключу          | `Dictionary[key]`                    |
| Проверка наличия / уникальности             | `Set.contains(_:)`                   |
| Группировка по полю                         | `Dictionary(grouping:by:)`           |
| Кэширование по URL / ID / token             | `Dictionary` или `NSCache`           |
| Данные не уникальны / нужен первый элемент  | `first(where:)`                      |
| Нужно сохранить порядок вставки             | `OrderedDictionary` (или сторонние)  |

В iOS-разработке **хэш-поиск через Dictionary и Set** — это **основа** большинства операций поиска и кэширования.  
Практически все задачи, связанные с быстрым доступом по ключу, решаются именно через них.
