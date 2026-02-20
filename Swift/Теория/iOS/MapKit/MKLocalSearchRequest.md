**`MKLocalSearchRequest`** — это устаревший класс в **MapKit**, который использовался для настройки локального поиска (POI, адресов, мест) до iOS 13.

Начиная с **iOS 13 (2019)**, Apple полностью заменила `MKLocalSearchRequest` на более современный и удобный класс **`MKLocalSearch.Request`**, который является **единственным рекомендуемым способом** выполнения локального поиска в 2026 году.

### Почему MKLocalSearchRequest устарел и когда его всё ещё можно встретить

| Статус (2026)          | Поддержка iOS / macOS | Рекомендация Apple | Где ещё можно увидеть |
|------------------------|-----------------------|---------------------|-----------------------|
| **Deprecated**         | iOS 13.0+             | Используйте `MKLocalSearch.Request` | Старый код, legacy-проекты, Stack Overflow до 2019–2020 |
| **Не удалён**          | Работает, но с предупреждением | —                   | — |
| **Альтернатива**       | `MKLocalSearch.Request` | **Обязательно**     | Все новые проекты     |

### Основные свойства MKLocalSearchRequest (для понимания legacy-кода)

| Свойство                     | Тип                          | Что делал (в старом API)                              | Эквивалент в MKLocalSearch.Request (новый) |
|------------------------------|------------------------------|-------------------------------------------------------|---------------------------------------------|
| `naturalLanguageQuery`       | `String?`                    | Поисковый запрос ("кофе рядом", "пицца Москва")       | То же самое                                 |
| `region`                     | `MKCoordinateRegion?`        | Ограничение области поиска                            | То же самое                                 |
| `pointOfInterestFilter`      | `MKPointOfInterestFilter?`   | Фильтр по категориям POI                              | То же самое                                 |
| `resultTypes`                | `MKSearchResultType`         | Типы результатов (адрес, POI, телефон)                | `resultTypes` (тот же)                      |

### Переход с MKLocalSearchRequest на MKLocalSearch.Request (как выглядит современный код)

**Старый (deprecated) код** (до iOS 13):

```swift
let request = MKLocalSearchRequest()
request.naturalLanguageQuery = "кофейня"
request.region = mapView.region

let search = MKLocalSearch(request: request)
search.start { response, error in
    // ...
}
```

**Современный код (рекомендуемый 2026)**:

```swift
let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "кофейня"
request.region = mapView.region
request.pointOfInterestFilter = MKPointOfInterestFilter(including: [.cafe, .restaurant])

let search = MKLocalSearch(request: request)
search.start { response, error in
    guard let response else {
        print("Ошибка поиска:", error?.localizedDescription ?? "Неизвестно")
        return
    }
    
    let mapItems = response.mapItems  // [MKMapItem]
    // Добавляем аннотации на карту
    let annotations = mapItems.map { item in
        let ann = MKPointAnnotation()
        ann.coordinate = item.placemark.coordinate
        ann.title = item.name
        ann.subtitle = item.phoneNumber ?? item.placemark.title
        return ann
    }
    
    mapView.addAnnotations(annotations)
}
```

### Лучшие практики (2026)

- **Никогда** не используйте `MKLocalSearchRequest` в новом коде — Xcode покажет предупреждение  
- **Всегда** используйте `MKLocalSearch.Request` — это единственный поддерживаемый способ  
- **Ограничивайте** `region` — поиск будет быстрее и релевантнее  
- **Используйте** `pointOfInterestFilter` — сильно уменьшает мусор в результатах  
- **Обрабатывайте** `error` и пустой `response` — показывайте пользователю сообщение  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKLocalSearch.Request — поиск кофеен в видимой области карты»

**Короткий итог 2026**:
> `MKLocalSearchRequest` — **устаревший** класс (deprecated с iOS 13), который раньше использовался для настройки поиска мест.  
> В 2026 году:  
> - полностью заменён на `MKLocalSearch.Request`  
> - все новые проекты используют только `MKLocalSearch.Request`  
> - функциональность идентична, но синтаксис чище и современнее  
> - если увидите `MKLocalSearchRequest` в коде — это legacy, обновите на `MKLocalSearch.Request`

Удачи с актуальным и эффективным поиском мест в твоём приложении! 🔍🗺️