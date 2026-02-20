**`MKMapItem`** — это класс в фреймворке **MapKit**, который представляет **одно место** (point of interest, POI) на карте Apple Maps.

Это **самый удобный и часто используемый** способ работы с конкретными локациями: адресами, организациями, точками интереса, результатами поиска и т.д.

### Основные свойства MKMapItem (актуально на 2026 год)

| Свойство                     | Тип                          | Описание / Пример значения                              | Самый частый сценарий |
|------------------------------|------------------------------|---------------------------------------------------------|------------------------|
| `placemark`                  | `CLPlacemark`                | Полная информация о месте (координаты, адрес, название) | Основной источник данных |
| `name`                       | `String?`                    | Название места ("Starbucks", "Красная площадь")         | Заголовок аннотации    |
| `phoneNumber`                | `String?`                    | Телефон в международном формате                         | Кнопка "Позвонить"     |
| `url`                        | `URL?`                       | Сайт места                                              | Кнопка "Открыть сайт"  |
| `coordinate`                 | `CLLocationCoordinate2D`     | Координаты (широта и долгота)                           | Добавление на карту    |
| `isCurrentLocation`          | `Bool`                       | Это ли текущее местоположение пользователя              | Отличие от других точек |
| `timeZone`                   | `TimeZone?`                  | Часовой пояс места                                      | Отображение времени    |
| `pointOfInterestCategory`    | `String?`                    | Категория POI (например, "Cafe", "Restaurant")          | Фильтрация/иконки      |

### Самые популярные способы получения MKMapItem

#### 1. Из результатов поиска (самый частый сценарий)

```swift
let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "кофейня рядом"
request.region = mapView.region

let search = MKLocalSearch(request: request)
search.start { response, error in
    guard let response else { return }
    
    let items: [MKMapItem] = response.mapItems
    
    // Добавляем все найденные места как аннотации
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

#### 2. Создание MKMapItem вручную (из координат или адреса)

```swift
// Из координат
let coordinate = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
let placemark = MKPlacemark(coordinate: coordinate)
let mapItem = MKMapItem(placemark: placemark)
mapItem.name = "Красная площадь"
mapItem.phoneNumber = "+7 (495) 123-45-67"

// Из адреса (через CLGeocoder)
let geocoder = CLGeocoder()
geocoder.geocodeAddressString("Москва, Тверская ул., 1") { placemarks, error in
    guard let placemark = placemarks?.first else { return }
    
    let mapItem = MKMapItem(placemark: placemark)
    mapItem.name = "Тверская, 1"
    
    // Открываем в Apple Maps
    mapItem.openInMaps(launchOptions: [MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving])
}
```

#### 3. Открытие маршрута в Apple Maps (очень популярно)

```swift
let start = MKMapItem.forCurrentLocation()
let end = MKMapItem(placemark: MKPlacemark(coordinate: destinationCoordinate))
end.name = "Цель поездки"

let launchOptions = [
    MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving,
    MKLaunchOptionsShowsTrafficKey: true
]

MKMapItem.openMaps(with: [start, end], launchOptions: launchOptions)
```

### Лучшие практики MKMapItem в Swift 2026

- **Всегда** берите `MKMapItem` из результатов `MKLocalSearch` или `CLGeocoder` — они содержат максимум полезных данных  
- **Используйте** `mapItem.openInMaps(launchOptions:)` — это стандартный способ показать маршрут в Apple Maps  
- **Для аннотаций** — создавайте `MKPointAnnotation` из `mapItem.placemark.coordinate` + `name`  
- **Для телефонов и сайтов** — проверяйте `phoneNumber` и `url` перед показом кнопок  
- **Для текущего местоположения** — используйте `MKMapItem.forCurrentLocation()`  
- **В SwiftUI** — передавайте `MKMapItem` в `Map` / `MapMarker` или используйте в `openMaps`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit и Core Location  
- **Документируйте** — пишите комментарий «MKMapItem — результат поиска кофейни с координатами и телефоном»

**Короткий итог 2026**:
> `MKMapItem` — это **полноценный объект места** на карте: координаты + название + адрес + телефон + сайт + категория.  
> В 2026 году:  
> - чаще всего получаете из `MKLocalSearch.Response.mapItems`  
> - используете для: аннотаций, открытия в Apple Maps, маршрутов, отображения деталей  
> - это **самый удобный** и **самый богатый данными** способ работать с конкретными локациями в iOS-приложениях  

Удачи с информативными и удобными точками интереса в твоём приложении! 📍☕️