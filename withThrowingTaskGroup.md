**withThrowingTaskGroup** — это один из самых мощных и часто используемых инструментов **структурированной конкурентности** в [[Swift]] (начиная с Swift 5.5 и особенно актуальный в Swift 6+).

Это функция высшего порядка, которая позволяет:

- запускать **несколько параллельных задач** (throwing)  
- собирать их результаты в группу  
- **автоматически отменять** все вложенные задачи при выходе из блока  
- **обрабатывать ошибки** ([[throw]]) из любой задачи  
- **ждать завершения всех** задач перед продолжением

### Основной синтаксис (2026 актуальный)

```swift
try await withThrowingTaskGroup(of: ResultType.self) { group in
    // добавляем задачи
    group.addTask {
        try await doSomethingAsync()
    }
    
    group.addTask {
        try await doAnotherThing()
    }
    
    // обрабатываем результаты по мере готовности
    for try await result in group {
        // result имеет тип ResultType
        print(result)
    }
}
```

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Параллельная загрузка нескольких ресурсов (самый частый сценарий)

```swift
struct ProfileData {
    let user: User
    let posts: [Post]
    let stories: [Story]
}

@MainActor
func loadProfile() async throws -> ProfileData {
    try await withThrowingTaskGroup(of: ProfilePart.self) { group in
        
        group.addTask {
            let user = try await api.fetchCurrentUser()
            return .user(user)
        }
        
        group.addTask {
            let posts = try await api.fetchPosts()
            return .posts(posts)
        }
        
        group.addTask {
            let stories = try await api.fetchStories()
            return .stories(stories)
        }
        
        var result = ProfileData(user: .empty, posts: [], stories: [])
        
        for try await part in group {
            switch part {
            case .user(let user):     result.user = user
            case .posts(let posts):   result.posts = posts
            case .stories(let stories): result.stories = stories
            }
        }
        
        return result
    }
}

private enum ProfilePart {
    case user(User)
    case posts([Post])
    case stories([Story])
}
```

#### 2. Параллельная обработка массива ([[map]] в параллель)

```swift
func processImages(_ urls: [URL]) async throws -> [UIImage] {
    try await withThrowingTaskGroup(of: (index: Int, image: UIImage).self) { group in
        
        for (index, url) in urls.enumerated() {
            group.addTask {
                let (data, _) = try await URLSession.shared.data(from: url)
                guard let image = UIImage(data: data) else {
                    throw ImageError.invalidData
                }
                return (index, image)
            }
        }
        
        var images = [UIImage?](repeating: nil, count: urls.count)
        
        for try await (index, image) in group {
            images[index] = image
        }
        
        return images.compactMap { $0 }
    }
}
```

#### 3. Отмена всех задач при ошибке одной (по умолчанию)

```swift
try await withThrowingTaskGroup(of: String.self) { group in
    group.addTask {
        try await fetchUserData()     // если здесь ошибка → все отменяются
    }
    
    group.addTask {
        try await fetchPosts()
    }
    
    // Первая ошибка отменит все остальные задачи автоматически
    var results: [String] = []
    for try await result in group {
        results.append(result)
    }
}
```

### Лучшие практики withThrowingTaskGroup в Swift 2026

- **Всегда используй throwing-версию**, если хотя бы одна задача может бросить ошибку  
- **Обрабатывай CancellationError** — обычно просто игнорируй (`catch is CancellationError { }`)  
- **Собирай результаты по мере готовности** — `for try await result in group`  
- **Используй индексы**, если порядок важен (как в примере с images)  
- **[[@MainActor]]** — если работаешь с UI внутри группы → оборачивай в `await MainActor.run`  
- **Swift 6 strict concurrency** — группа полностью безопасна, но результаты должны быть Sendable  
- **Не забывай try** — группа бросает первую ошибку, остальные задачи отменяются  
- **Документируйте** — пиши комментарий «withThrowingTaskGroup — параллельная загрузка с автоматической отменой»

**Короткий девиз 2026**:
> «withThrowingTaskGroup — это когда ты хочешь запустить несколько параллельных задач, собрать их результаты, автоматически отменить всё при ошибке или выходе из scope и не думать о ручной отмене.  
> В 2026 году это **основной** инструмент для параллельной загрузки данных, обработки массивов и конкурентных операций.  
> Забудь [[DispatchGroup]] — используй withThrowingTaskGroup.»
