**`MKPointOfInterestFilter`** — это класс в **MapKit** (доступен с iOS 13.0+), который позволяет **фильтровать** результаты поиска по категориям точек интереса (POI — Points of Interest) при использовании **`MKLocalSearch`**.

С его помощью можно:
- включить только определённые категории (например, только кафе и рестораны),
- исключить нежелательные категории (например, убрать все магазины),
- комбинировать несколько фильтров.

Это **самый эффективный** способ сделать поиск релевантным и уменьшить количество мусора в результатах.

### Основные способы создания MKPointOfInterestFilter

#### 1. Включить только нужные категории (самый частый)

```swift
let filter = MKPointOfInterestFilter(including: [
    .cafe,
    .restaurant,
    .bakery,
    .coffeeShop
])

let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "кофе"
request.pointOfInterestFilter = filter
```

#### 2. Исключить ненужные категории

```swift
let filter = MKPointOfInterestFilter(excluding: [
    .store,
    .gasStation,
    .parking
])

request.pointOfInterestFilter = filter
```

#### 3. Комбинированный фильтр (включаем одни, исключаем другие)

```swift
// Только еда и напитки, но без фастфуда и баров
let filter = MKPointOfInterestFilter(including: [
    .cafe, .restaurant, .bakery
], excluding: [
    .fastFoodRestaurant, .bar, .nightlife
])
```

#### 4. Самый популярный реальный пример (поиск кафе/ресторанов в видимой области)

```swift
let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "еда"  // или "restaurant", "cafe"
request.region = mapView.region

// Фильтруем только нужные категории
request.pointOfInterestFilter = MKPointOfInterestFilter(including: [
    .cafe,
    .restaurant,
    .bakery,
    .foodMarket
])

let search = MKLocalSearch(request: request)
search.start { response, error in
    guard let response else { return }
    
    let items = response.mapItems
    
    // Добавляем на карту только отфильтрованные результаты
    let annotations = items.map { item -> MKPointAnnotation in
        let ann = MKPointAnnotation()
        ann.coordinate = item.coordinate
        ann.title = item.name
        ann.subtitle = item.phoneNumber ?? item.placemark.title
        return ann
    }
    
    mapView.addAnnotations(annotations)
}
```

### Все основные категории POI (самые используемые в 2026)

Полный список: https://developer.apple.com/documentation/mapkit/mkpointofinterestcategory

Самые популярные:

- `.cafe`, `.restaurant`, `.bakery`, `.foodMarket`
- `.fastFoodRestaurant`, `.bar`, `.nightlife`
- `.hotel`, `.motel`
- `.museum`, `.artGallery`, `.landmark`
- `.park`, `.beach`, `.playground`
- `.university`, `.school`
- `.airport`, `.publicTransport`
- `.store`, `.grocery`, `.pharmacy`
- `.gasStation`, `.parking`
- `.hospital`, `.pharmacy`

### Лучшие практики MKPointOfInterestFilter в Swift 2026

- **Всегда** применяйте фильтр — без него поиск возвращает слишком много мусора (особенно при общих запросах типа "еда", "магазин")  
- **Используйте** `including:` для точного поиска (например, только кафе)  
- **Используйте** `excluding:` для удаления нежелательных категорий (например, убрать АЗС и парковки)  
- **Комбинируйте** оба — это даёт максимальную точность  
- **Ограничивайте** `region` — фильтр работает только внутри указанной области  
- **Для автодополнения** — `MKLocalSearchCompleter` тоже поддерживает `pointOfInterestFilter` (с iOS 13+)  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKPointOfInterestFilter — только кафе и рестораны в видимой области карты»

**Короткий итог 2026**:
> `MKPointOfInterestFilter` — это **фильтр категорий** для поиска мест в `MKLocalSearch`.  
> В 2026 году:  
> - создаётся через `including:` / `excluding:` или их комбинацию  
> - передаётся в `MKLocalSearch.Request.pointOfInterestFilter`  
> - основные категории — `.cafe`, `.restaurant`, `.bakery`, `.hotel`, `.museum`, `.park` и т.д.  
> - сильно улучшает релевантность результатов и уменьшает мусор  
> Это **обязательный** инструмент для любого качественного поиска по карте в iOS-приложении.

Удачи с точным и релевантным поиском мест в твоём приложении! 🔍🍽️