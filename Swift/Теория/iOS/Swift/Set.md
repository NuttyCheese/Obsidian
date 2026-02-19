**`Set`** — это **неупорядоченная коллекция уникальных элементов** в Swift.

Ключевые характеристики (актуально на 2026 год):

- **Уникальность** — дубликаты автоматически игнорируются
- **Порядок не гарантирован** — элементы хранятся в хеш-таблице (open addressing)
- **Элементы должны быть Hashable** (Int, String, UUID, struct/enum с Hashable и т.д.)
- **Средняя сложность операций** — O(1) для insert, remove, contains (при хорошем hash)
- **Mutable** через `var`, **immutable** через `let`
- **Value type** — копируется при присваивании (Copy-on-Write)

### Основные операции и методы (самые используемые в 2026)

| Операция / Метод                  | Синтаксис / Пример                                           | Сложность | Комментарий 2026 |
|-----------------------------------|--------------------------------------------------------------|-----------|------------------|
| Создание                          | `Set([1, 2, 3])` или `Set<Int>()`                            | —         | —                |
| Добавление                        | `set.insert(42)`                                             | O(1)      | Возвращает `(inserted: Bool, memberAfterInsert: T)` |
| Удаление                          | `set.remove(42)`                                             | O(1)      | Возвращает удалённое значение или nil |
| Проверка наличия                  | `set.contains(42)`                                           | O(1)      | Самый быстрый способ проверки |
| Объединение (union)               | `a.union(b)` или `a.formUnion(b)` (in-place)                 | O(n)      | Возвращает новый Set или меняет существующий |
| Пересечение (intersection)        | `a.intersection(b)`                                          | O(n)      | Элементы, которые есть в обоих |
| Вычитание (subtracting)           | `a.subtracting(b)`                                           | O(n)      | Элементы, которые есть в a, но нет в b |
| Симметричная разность             | `a.symmetricDifference(b)`                                   | O(n)      | Элементы, которые есть только в одном множестве |
| Пустота                           | `set.isEmpty`                                                | O(1)      | —                |
| Количество элементов              | `set.count`                                                  | O(1)      | —                |
| Итерация                          | `for item in set { ... }`                                    | O(n)      | Порядок случайный |
| map / filter / reduce             | `set.map { $0 * 2 }` → возвращает **Array**                  | O(n)      | Set → Array после трансформации |
| sorted                            | `set.sorted()`                                               | O(n log n)| Возвращает Array |

### Самые популярные паттерны Set в 2026 году

#### 1. Удаление дубликатов из массива (самый частый кейс)

```swift
let duplicates = [1, 2, 2, 3, 3, 4, 1]
let unique = Set(duplicates)          // → Set<Int> с уникальными значениями
let sortedUnique = Array(unique).sorted()  // если нужен порядок
```

#### 2. Быстрая проверка принадлежности (O(1) вместо O(n))

```swift
let allowedUsers: Set<String> = ["alice", "bob", "charlie"]

func canAccess(_ username: String) -> Bool {
    allowedUsers.contains(username.lowercased())
}
```

#### 3. Операции над множествами (очень полезно в бизнес-логике)

```swift
let admins: Set<String> = ["alice", "bob"]
let moderators: Set<String> = ["bob", "charlie"]

let allPrivileged = admins.union(moderators)              // {"alice", "bob", "charlie"}
let bothRoles = admins.intersection(moderators)           // {"bob"}
let onlyAdmins = admins.subtracting(moderators)           // {"alice"}
let xorRoles = admins.symmetricDifference(moderators)     // {"alice", "charlie"}
```

#### 4. Set с кастомными типами (Hashable обязателен)

```swift
struct UserID: Hashable {
    let value: String
}

let activeUsers: Set<UserID> = [UserID(value: "u123"), UserID(value: "u456")]
```

#### 5. Set + lazy / функциональные методы

```swift
let numbers: Set = [1, 2, 3, 4, 5, 6]

let evenSquared = numbers.lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * $0 }
    .sorted()

// Результат — массив [4, 16, 36]
```

### 5. Лучшие практики Set в Swift 2026

- **Используй Set** именно тогда, когда тебе нужна **уникальность** + **быстрая проверка contains**  
- **Не используй Set**, если важен порядок → бери `Array` или `OrderedSet` (swift-collections)  
- **Не используй Set**, если часто нужен индекс → `Array` быстрее  
- **Для очень больших множеств** — рассмотри `Set` с хорошим `Hashable` (плохой hash → деградация до O(n))  
- **В SwiftUI** — Set идеален для хранения выбранных ID (`Set<UUID>`)  
- **Swift 6 strict concurrency** — `Set` полностью `Sendable` при `Element: Sendable`  
- **Документируйте** — пиши комментарий «Set<UserID> — активные пользователи (уникальные ID)»

**Короткий девиз 2026**:
> `Set` — это когда тебе нужны **уникальные элементы** и **мгновенная проверка наличия**.  
> В 2026 году:  
> - удаление дубликатов → `Set(array)`  
> - быстрая проверка `contains` → `Set` вместо `Array.contains`  
> - операции над множествами (union, intersection и т.д.)  
> - для порядка → `Array(sorted(set))` или `OrderedSet`  
> Это **самый быстрый** способ гарантировать уникальность в Swift.

Удачи с быстрыми и уникальными коллекциями в твоём коде! 🚀