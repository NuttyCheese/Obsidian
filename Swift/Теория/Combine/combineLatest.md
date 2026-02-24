**`combineLatest`** — один из самых мощных и часто используемых операторов в **[[Combine]]** (и в [[RxSwift]]/RxCocoa).

Он объединяет **несколько [[Publisher]]** в один, выдавая **новое значение каждый раз, когда любой из источников испускает новое значение**, при этом беря **последние известные значения** всех остальных источников.

### Ключевые особенности combineLatest (2025–2026)

| Характеристика               | Описание                                                                     | Важно помнить                                       |
| ---------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------- |
| Количество входных Publisher | От 2 до 4 (перегруженные методы) или коллекция через `combineLatest(_:)`     | >4 — используй `Publishers.MergeMany` или `Zip`     |
| Выдаёт значение              | Только когда **все** Publisher хотя бы раз испустили значение                | Если один Publisher ещё не начал — ничего не придёт |
| Последние значения           | Всегда берёт **самое свежее** значение каждого Publisher                     | Идеально для форм (email + пароль + согласие)       |
| Завершение                   | Завершается, когда **любой** из Publisher завершается с ошибкой или finished | `.merge` завершается только когда все завершатся    |
| Ошибки                       | Если любой Publisher выдаёт ошибку — весь поток падает                       | Используй `.catch` или `.replaceError`              |
| Backpressure                 | Учитывает demand от [[Subscriber]]                                           | Работает корректно с `buffer` и `demand`            |

### Когда использовать combineLatest (реальные кейсы 2026)

| Сценарий                                      | Почему именно combineLatest                                          | Альтернатива |
|-----------------------------------------------|-----------------------------------------------------------------------|--------------|
| Валидация формы (email + пароль + согласие)   | Нужно знать актуальное состояние всех полей одновременно              | `withLatestFrom` (Rx) |
| Отображение полного имени (firstName + lastName) | Обновлять лейбл только когда меняется любое из полей                  | `map` + `zip` |
| Синхронизация нескольких асинхронных запросов | Дождаться всех ответов и объединить их последние значения             | `zip` (если нужен ровно один раз) |
| Показ индикатора "всё загружено"              | Комбинировать несколько потоков загрузки                              | `combineLatest` + `map` |
| Реактивный UI с несколькими источниками       | Температура + влажность + статус сети → обновлять UI                  | — |

### Самые популярные и рекомендуемые примеры (2026 стиль)

#### 1. Классическая валидация формы (самый частый кейс)

```swift
class LoginViewModel: ObservableObject {
    
    @Published var email = ""
    @Published var password = ""
    @Published var isAgreed = false
    @Published var isLoginEnabled = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        Publishers.CombineLatest3($email, $password, $isAgreed)
            .map { email, password, agreed in
                !email.isEmpty &&
                password.count >= 6 &&
                email.contains("@") &&
                agreed
            }
            .assign(to: &$isLoginEnabled)
    }
}
```

В контроллере:

```swift
viewModel.$isLoginEnabled
    .assign(to: \.isEnabled, on: loginButton)
    .store(in: &cancellables)
```

#### 2. Объединение нескольких сетевых запросов

```swift
struct UserProfile {
    let user: User
    let posts: [Post]
    let settings: Settings
}

func fetchProfile() -> AnyPublisher<UserProfile, Error> {
    Publishers.CombineLatest3(
        fetchUser(),
        fetchPosts(),
        fetchSettings()
    )
    .map { user, posts, settings in
        UserProfile(user: user, posts: posts, settings: settings)
    }
    .eraseToAnyPublisher()
}

fetchProfile()
    .receive(on: DispatchQueue.main)
    .sink { completion in
        // обработка ошибок
    } receiveValue: { profile in
        // отобразить всё сразу
    }
    .store(in: &cancellables)
```

#### 3. Отложенное обновление UI с debounce

```swift
$searchText
    .debounce(for: .seconds(0.4), scheduler: DispatchQueue.main)
    .combineLatest($filterCategory)
    .map { text, category in
        filterItems(text: text, category: category)
    }
    .assign(to: &$filteredItems)
```

#### 4. combineLatest с коллекцией Publisher (iOS 14+)

```swift
let publishers = urls.map { URLSession.shared.dataTaskPublisher(for: $0).map(\.data) }

Publishers.MergeMany(publishers)
    .collect()
    .sink { dataArray in
        // обработать все данные
    }
    .store(in: &cancellables)
```

### Сравнение combineLatest с другими операторами

| Оператор          | Когда выдаёт значение                              | Кол-во значений | Когда использовать в 2026 |
|-------------------|-----------------------------------------------------|------------------|----------------------------|
| `combineLatest`   | Когда **любой** источник выдаёт значение            | Бесконечно       | Формы, синхронизация состояний |
| `zip`             | Когда **все** источники выдали по одному значению   | Ограничено       | Параллельные запросы (первый раз) |
| `merge`           | Когда любой источник выдаёт значение                | Бесконечно       | Объединение событий без ожидания |
| `withLatestFrom`  | Когда основной источник выдаёт, берёт последнее из другого | Бесконечно       | Rx-стиль (в Combine нет прямого аналога) |

### Лучшие практики combineLatest в 2026 году

- **Всегда** используйте `CombineLatest` / `CombineLatest3` / `CombineLatest4` — для >4 используйте `Publishers.MergeMany` или кастомный оператор  
- **Держите** цепочку **короткой** — если >3–4 источников, разбейте на промежуточные `@Published`  
- **Используйте** `.debounce` перед `combineLatest`, если источники — UI-ввод  
- **Обязательно** добавляйте `.receive(on: DispatchQueue.main)` перед `.assign` / `.sink` для UI  
- **Для ошибок** — используйте `.catch` или `.replaceError` до `combineLatest`  
- **Для тестирования** — используйте `Publishers.Just`, `PassthroughSubject`, `CurrentValueSubject`  
- **Документируйте** — пишите комментарий:

```swift
// Валидация формы: email + пароль + согласие → кнопка активна
Publishers.CombineLatest3($email, $password, $isAgreed)
    .map { ... }
    .assign(to: &$isLoginEnabled)
```

**Короткий итог 2026**:
> `combineLatest` — оператор, который **объединяет последние значения** нескольких Publisher и выдаёт новый tuple каждый раз, когда любой из них обновляется.  
> В 2026 году:  
> - самый популярный кейс — валидация форм, синхронизация состояний  
> - используйте `CombineLatest3` / `CombineLatest4` для читаемости  
> - всегда `.debounce` для UI-ввода и `.receive(on: .main)` для UI-обновлений  
> - это **один из самых мощных** операторов Combine для реактивного UI  
