**`MKCoordinateRegion`** — это структура в **[[MapKit]]**, которая определяет **видимую область карты** в [[MKMapView]].

Она описывает:
- **центр** области (координаты широты и долготы),
- **размер** видимой области (в градусах широты и долготы — **span**).

Это **основной способ** задавать, какую часть карты показывает `MKMapView` при вызове `setRegion(_:animated:)`.

### Основные свойства структуры MKCoordinateRegion

| Свойство | Тип                        | Описание                                    | Значение по умолчанию / пример |
| -------- | -------------------------- | ------------------------------------------- | ------------------------------ |
| `center` | [[CLLocationCoordinate2D]] | Центральная точка видимой области           | Обязательно задавать           |
| `span`   | [[MKCoordinateSpan]]       | Размер области (ширина и высота в градусах) | Обязательно задавать           |

Внутренняя структура `MKCoordinateSpan`:

| Свойство         | Тип                   | Что означает                              | Типичный диапазон значений     |
| ---------------- | --------------------- | ----------------------------------------- | ------------------------------ |
| `latitudeDelta`  | [[CLLocationDegrees]] | Разница широты (север-юг) в градусах      | 0.001 (город) … 180 (весь мир) |
| `longitudeDelta` | `CLLocationDegrees`   | Разница долготы (запад-восток) в градусах | 0.001 (город) … 360 (весь мир) |

### Самые популярные способы создания MKCoordinateRegion

#### 1. Ручное создание (самый частый)

```swift
let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва, Красная площадь
let span = MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)     // ~5–6 км в диаметре

let region = MKCoordinateRegion(center: center, span: span)

mapView.setRegion(region, animated: true)
```

#### 2. Через метры (самый удобный и рекомендуемый в 2026)

```swift
let center = CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194) // Сан-Франциско
let region = MKCoordinateRegion(
    center: center,
    latitudinalMeters: 2000,   // ±1 км по широте
    longitudinalMeters: 2000   // ±1 км по долготе (зависит от широты)
)

mapView.setRegion(region, animated: true)
```

#### 3. Центрирование на маршруте / нескольких точках

```swift
let points = [startCoordinate, endCoordinate] // или все точки маршрута
let rect = MKMapRect(points: points) // или route.boundingMapRect

mapView.setVisibleMapRect(rect, 
                         edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                         animated: true)
```

#### 4. Центрирование на пользователе с запасом

```swift
guard let userLocation = mapView.userLocation.location else { return }

let region = MKCoordinateRegion(
    center: userLocation.coordinate,
    latitudinalMeters: 1000,
    longitudinalMeters: 1000
)

mapView.setRegion(region, animated: true)
```

### Лучшие практики MKCoordinateRegion в Swift 2026

- **Предпочитайте** `latitudinalMeters` / `longitudinalMeters` — это гораздо понятнее и независимо от широты  
- **Не используйте** слишком маленькие `delta` (< 0.001) — карта может "зависнуть" или показывать шум  
- **Для маршрутов** — всегда берите `route.boundingMapRect` и добавляйте `edgePadding`  
- **Для динамического зума** — используйте `MKMapViewCameraZoomRange` (iOS 13+)  
- **Для 3D-вида** — используйте `MKMapCamera` вместо региона  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и устанавливайте регион в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKCoordinateRegion — видимая область карты ±1 км вокруг центра»

**Короткий итог 2026**:
> `MKCoordinateRegion` — это **видимая область карты** (центр + размер в градусах или метрах).  
> В 2026 году:  
> - создаётся через `MKCoordinateRegion(center:span:)` или `latitudinalMeters/longitudinalMeters`  
> - задаётся через `mapView.setRegion(_:animated:)`  
> - самый удобный способ — метры (`latitudinalMeters`, `longitudinalMeters`)  
> - для маршрутов — используйте `boundingMapRect` + `setVisibleMapRect`  
> Это **основной** способ управлять тем, что именно видит пользователь на карте.
