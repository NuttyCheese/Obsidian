**`MKDirectionsTransportType`** — это перечисление ([[enum]]) в фреймворке **[[MapKit]]**, которое определяет **тип транспорта**, используемый при построении маршрута через **[[MKDirections]]** и **`MKDirections.Request`**.

Оно задаётся в свойстве `transportType` запроса и влияет на то, какие маршруты будет искать Apple Maps.

### Все возможные значения (актуально на 2026 год)

| Значение в Swift                        | Константа в Objective-C                  | Описание маршрута                                      | Когда использовать (самые частые сценарии) |
|-----------------------------------------|------------------------------------------|--------------------------------------------------------|---------------------------------------------|
| `.automobile`                           | `MKDirectionsTransportTypeAutomobile`    | Автомобильный маршрут (с учётом дорог, пробок, платных участков) | Навигация на машине, такси, каршеринг       |
| `.walking`                              | `MKDirectionsTransportTypeWalking`       | Пешеходный маршрут (тротуары, пешеходные дорожки)      | Прогулки, пешие маршруты, короткие расстояния |
| `.transit`                              | `MKDirectionsTransportTypeTransit`       | Общественный транспорт (метро, автобусы, электрички)   | Городская навигация без машины              |
| `.any`                                  | `MKDirectionsTransportTypeAny`           | Любой доступный тип (авто + пеший + транспорт)         | Когда тип не важен, максимум вариантов      |

**Важно**:  
- `.cycling` (велосипедный транспорт) **отсутствует** в публичном [[API]] [[MapKit]] (по состоянию на 2026 год).  
  Apple Maps поддерживает велосипедные маршруты в некоторых странах, но в [[MKDirections]] это пока недоступно.

### Самые популярные паттерны использования MKDirectionsTransportType

#### 1. Автомобильный маршрут (самый частый)

```swift
let request = MKDirections.Request()
request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
request.transportType = .automobile          // ← основной выбор
request.requestsAlternateRoutes = true       // альтернативные варианты

let directions = MKDirections(request: request)
directions.calculate { response, error in
    guard let route = response?.routes.first else { return }
    mapView.addOverlay(route.polyline)
}
```

#### 2. Пешеходный маршрут (очень популярен для коротких расстояний)

```swift
request.transportType = .walking
request.requestsAlternateRoutes = false  // альтернативы редко нужны для пешего
```

#### 3. Общественный транспорт (городская навигация)

```swift
request.transportType = .transit
// Можно указать дату/время отправления (опционально)
request.departureDate = Date() + 3600  // через час
```

#### 4. Универсальный поиск (любой транспорт)

```swift
request.transportType = .any
// Получаем все возможные варианты (авто + пеший + транспорт)
```

### Как отобразить несколько типов транспорта (часто используемый паттерн)

```swift
func showAllTransportOptions(from start: CLLocationCoordinate2D, to end: CLLocationCoordinate2D) {
    let types: [MKDirectionsTransportType] = [.automobile, .walking, .transit]
    
    for type in types {
        let request = MKDirections.Request()
        request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
        request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
        request.transportType = type
        
        let directions = MKDirections(request: request)
        directions.calculate { response, error in
            guard let route = response?.routes.first else { return }
            
            let polyline = route.polyline
            polyline.title = type == .automobile ? "Авто" : type == .walking ? "Пешком" : "Транспорт"
            
            self.mapView.addOverlay(polyline)
        }
    }
}
```

### Лучшие практики MKDirectionsTransportType в Swift 2026

- **По умолчанию** — используйте `.automobile` — это самый надёжный и востребованный тип  
- **Для пеших маршрутов** — всегда `.walking` (особенно для коротких расстояний < 5 км)  
- **Для города** — `.transit` даёт реальные расписания и варианты пересадок  
- **Не используйте** `.any` в продакшене без необходимости — результаты могут быть слишком разными и запутанными  
- **Учитывайте** `departureDate` / `arrivalDate` для `.transit` — без даты поиск может быть неточным  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и запускайте `MKDirections` в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «transportType = .automobile — автомобильный маршрут с учётом пробок»

**Короткий итог 2026**:
> `MKDirectionsTransportType` — это **тип транспорта** для расчёта маршрута в `MKDirections.Request`.  
> В 2026 году:  
> - основные варианты — `.automobile` (по умолчанию), `.walking`, `.transit`  
> - `.any` — редко, только для максимального количества вариантов  
> - велосипедные маршруты — **пока недоступны** в публичном API  
> Это **ключевой** параметр, который определяет, какой именно маршрут построит Apple Maps.
