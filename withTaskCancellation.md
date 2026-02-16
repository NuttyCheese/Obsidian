**withTaskCancellation** — это один из самых полезных и часто используемых инструментов в **[[Swift Concurrency]]** (начиная с [[Swift]] 5.5 и особенно актуальный в Swift 6+).

Это **структурированный** способ отменять задачи ([[Task]]) при выходе из области видимости (scope).

### Основная идея одним предложением

```swift
withTaskCancellation(id: someID, cancel: { ... }) {
    // здесь все вложенные Task автоматически отменяются, 
    // если мы вышли из этого блока (return, throw, конец scope)
}
```

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Самый частый случай: отмена всех вложенных задач при уходе с экрана

```swift
@MainActor
class ProfileViewController: UIViewController {
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        // Все задачи внутри этого блока будут отменены автоматически,
        // когда viewDidDisappear / deinit / уход с экрана
        withTaskCancellation(id: "profile-loading") {
            Task {
                do {
                    try await loadFullProfile()
                } catch is CancellationError {
                    print("Загрузка профиля отменена")
                } catch {
                    print("Ошибка: \(error)")
                }
            }
            
            Task {
                await loadPosts()
            }
            
            Task {
                await loadStories()
            }
        }
    }
}
```

#### 2. Явная отмена по уникальному ID (часто для поиска / фильтров)

```swift
@MainActor
class SearchViewModel: ObservableObject {
    
    @Published var results: [SearchResult] = []
    private var currentSearchID = UUID()
    
    func search(query: String) {
        // Предыдущий поиск отменяется автоматически
        withTaskCancellation(id: currentSearchID) {
            currentSearchID = UUID()  // новый уникальный ID
            
            Task {
                do {
                    let newResults = try await searchService.search(query: query)
                    await MainActor.run {
                        self.results = newResults
                    }
                } catch is CancellationError {
                    // игнорируем, это нормально
                } catch {
                    await MainActor.run {
                        // показать ошибку
                    }
                }
            }
        }
    }
}
```

#### 3. Отмена при смене состояния (switch / tab / фильтр)

```swift
@MainActor
class FeedViewModel {
    
    enum FeedType {
        case forYou, following
    }
    
    @Published var currentType: FeedType = .forYou
    
    func loadFeed() {
        withTaskCancellation(id: currentType) {
            Task {
                let posts = try await feedService.fetch(for: currentType)
                await MainActor.run {
                    self.posts = posts
                }
            }
        }
    }
    
    func changeFeed(to type: FeedType) {
        currentType = type
        loadFeed()  // предыдущая задача отменяется автоматически
    }
}
```

### Лучшие практики withTaskCancellation в Swift 2026

- **Используй уникальный ID** ([[UUID]], [[enum]], [[String]] с hash) — чтобы не отменять чужие задачи  
- **Помещай в самый узкий scope** — например, внутри [[viewWillAppear]] → [[viewDidDisappear]]  
- **Обрабатывай CancellationError** — обычно просто игнорируй (`catch is CancellationError { }`)  
- **Не забывай [[@MainActor]]** — если работаешь с UI внутри задачи  
- **Swift 6 strict concurrency** — withTaskCancellation полностью безопасен и проверяется компилятором  
- **Не используй Task.cancel() вручную** — withTaskCancellation делает это автоматически и структурировано  
- **Документируйте** — пиши комментарий «withTaskCancellation — все вложенные задачи отменяются при смене состояния / уходе с экрана»

**Короткий девиз 2026**:
> «withTaskCancellation — это когда ты говоришь: «все задачи внутри этого блока должны умереть, как только мы вышли из scope».  
> В 2026 году это **самый чистый**, **самый безопасный** и **самый рекомендуемый** способ отменять сетевые запросы, загрузки и длительные операции при уходе с экрана, смене фильтра или вкладки.  
> Забудь Task.cancel() вручную — используй withTaskCancellation.»
