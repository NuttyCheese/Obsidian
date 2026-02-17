**SwiftData** — это современный фреймворк от Apple для работы с локальными данными на [[iOS]], iPadOS, macOS, watchOS и visionOS, представленный на WWDC 2023 и ставший основным выбором для новых проектов к 2026 году.

Он полностью заменяет **[[Core Data]]** в большинстве случаев для приложений, которые не требуют очень сложных миграций или огромных объёмов данных.

### Ключевые преимущества SwiftData (2026)

| Характеристика            | SwiftData (2024–2026)                                   | Core Data (классика)                                   | Кто побеждает в 2026      |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------------ | ------------------------- |
| API                       | Декларативный, Swift-native (@Model, @Query)            | Императивный, NSManagedObject                          | **SwiftData**             |
| Интеграция со [[SwiftUI]] | Нативная, @Query, @ModelContext                         | Требует FetchRequest / NSFetchedResultsController      | **SwiftData**             |
| Миграции схемы            | Автоматические + лёгкие ручные                          | Тяжёлые, с mapping model                               | **SwiftData**             |
| Производительность        | Очень хорошая (на базе Core Data под капотом)           | Хорошая, но больше boilerplate                         | Ничья                     |
| Поддержка CloudKit        | Полная (синхронизация одним свойством)                  | Требует ручной настройки NSPersistentCloudKitContainer | **SwiftData**             |
| Concurrency               | Полностью [[Sendable]] + [[actor]]-safe                 | Требует mainContext / perform                          | **SwiftData**             |
| Объём кода                | Минимальный                                             | Большой                                                | **SwiftData**             |
| Сложные отношения / fetch | Хорошая поддержка, но не такая гибкая как [[Core Data]] | Максимальная гибкость                                  | Core Data (редкие случаи) |
| Поддержка Apple           | Активно развивается (новые фичи каждый год)             | Поддерживается, но без новых фич                       | **SwiftData**             |

### Основные строительные блоки SwiftData (2026)

```swift
import SwiftData

// 1. Модель данных
@Model
final class TodoItem {
    @Attribute(.unique) var id: UUID = UUID()
    var title: String
    var isCompleted: Bool = false
    var priority: Int = 0
    var dueDate: Date?
    
    // Отношения
    @Relationship(deleteRule: .cascade, inverse: \TodoList.items)
    var list: TodoList?
    
    init(title: String, priority: Int = 0, dueDate: Date? = nil) {
        self.title = title
        self.priority = priority
        self.dueDate = dueDate
    }
}

@Model
final class TodoList {
    var name: String
    @Relationship(deleteRule: .nullify, inverse: \TodoItem.list)
    var items: [TodoItem] = []
    
    init(name: String) {
        self.name = name
    }
}
```

### Самые популярные паттерны 2026 года

#### 1. Простой список с @Query

```swift
struct TodoListView: View {
    @Query(sort: \TodoItem.priority, order: .reverse)
    private var todos: [TodoItem]
    
    @Environment(\.modelContext) private var context
    
    var body: some View {
        List {
            ForEach(todos) { todo in
                HStack {
                    Text(todo.title)
                    Spacer()
                    if todo.isCompleted {
                        Image(systemName: "checkmark.circle.fill")
                            .foregroundStyle(.green)
                    }
                }
                .swipeActions {
                    Button("Delete", role: .destructive) {
                        context.delete(todo)
                    }
                }
            }
        }
        .toolbar {
            Button("Add") {
                let new = TodoItem(title: "New Task")
                context.insert(new)
            }
        }
    }
}
```

#### 2. Фильтрация + сортировка + предикаты

```swift
@Query(filter: #Predicate<TodoItem> { $0.isCompleted == false },
       sort: [\SortDescriptor(\.dueDate, order: .forward),
              \SortDescriptor(\.priority, order: .reverse)])
private var activeTodos: [TodoItem]
```

#### 3. Фоновая синхронизация с CloudKit

```swift
let container = ModelContainer(for: TodoItem.self, TodoList.self,
                               configurations: ModelConfiguration(cloudKitContainerIdentifier: "iCloud.com.example.TodoApp"))
```

### Лучшие практики SwiftData в Swift 2026

- **@Model** — всегда final class (рекомендуется Apple)  
- **@Attribute(.unique)** — для id / email / других уникальных полей  
- **@Relationship** — всегда указывай deleteRule (.nullify / .cascade / .deny)  
- **@Query** — основной способ получения данных в SwiftUI  
- **@ModelContext** — для вставки/удаления/обновления  
- **CloudKit** — включай только если нужна синхронизация (добавляет ~30–50% overhead)  
- **Swift 6 strict concurrency** — SwiftData полностью Sendable-safe при использовании @MainActor  
- **Миграции** — автоматические в большинстве случаев; для сложных — VersionedSchema + MigrationPlan  
- **Тестирование** — используй in-memory container (`ModelConfiguration(isStoredInMemoryOnly: true)`)  
- **Документируйте** — пиши комментарий «@Model — SwiftData модель с CloudKit-синхронизацией»

**Короткий девиз 2026**:
> «SwiftData — это когда тебе нужна современная, декларативная, SwiftUI-native локальная база с автоматическими миграциями и опциональной CloudKit-синхронизацией.  
> В 2026 году это **основной** выбор для новых приложений вместо Core Data.  
> Core Data оставь только для очень сложных legacy-проектов или если нужна максимальная гибкость fetch-запросов.»
