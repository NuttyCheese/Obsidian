**`MKMapRect`** — это структура в **MapKit**, которая представляет **прямоугольную область** на карте в **плоской проекции Mercator** (в метрах).

Это **низкоуровневый** тип координат, который MapKit использует внутри для всех расчётов bounding box, видимой области, оверлеев, аннотаций и кластеризации.  
В отличие от `MKCoordinateRegion` (центр + span в градусах), `MKMapRect` работает в **метрах** и используется для точных геометрических операций.

### Основные свойства структуры MKMapRect

```swift
public struct MKMapRect {
    public var origin: MKMapPoint     // левый нижний угол прямоугольника
    public var size:   MKMapSize      // ширина (x) и высота (y) в метрах
}
```

Внутренние структуры:
- `MKMapPoint` — точка (x, y) в метрах
- `MKMapSize` — размер (width, height) в метрах

### Самые важные и часто используемые преобразования и методы

#### 1. Создание MKMapRect из координат (самый частый)

```swift
let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
let mapPoint = MKMapPoint(center)

// Прямоугольник 2 км × 2 км вокруг центра
let size = MKMapSize(width: 2000, height: 2000)
let rect = MKMapRect(origin: mapPoint, size: size)
```

#### 2. Самый популярный способ — из видимой области карты

```swift
// Получить текущую видимую прямоугольную область
let visibleRect = mapView.visibleMapRect

// Центрировать карту на этом rect с отступами
mapView.setVisibleMapRect(visibleRect,
                         edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                         animated: true)
```

#### 3. Из маршрута / оверлея / аннотаций (очень частый)

```swift
// Из маршрута (MKDirections)
let routeRect = route.polyline.boundingMapRect

// Из набора аннотаций
let annotations = mapView.annotations
let rect = MKMapRect(annotations: annotations)  // или вручную через reduce

mapView.setVisibleMapRect(rect, edgePadding: .init(top: 80, left: 80, bottom: 80, right: 80), animated: true)
```

#### 4. Проверка пересечения / включения точки

```swift
let point = MKMapPoint(coordinate)
if rect.contains(point) {
    print("Точка внутри прямоугольника")
}

if rect.intersects(otherRect) {
    print("Прямоугольники пересекаются")
}
```

### Лучшие практики MKMapRect в Swift 2026

- **Предпочитайте** `setVisibleMapRect(_:edgePadding:animated:)` — это самый точный и удобный способ центрировать карту под контент  
- **Используйте** `edgePadding` — добавляет отступы (в пикселях), чтобы аннотации/оверлеи не прижимались к краям экрана  
- **Для маршрутов** — всегда берите `route.polyline.boundingMapRect` — это уже готовый оптимальный прямоугольник  
- **Для аннотаций** — используйте `MKMapRect(annotations:)` или вручную через `reduce` с `union`  
- **Для производительности** — избегайте частых пересчётов `boundingMapRect` — кэшируйте результат  
- **В SwiftUI** — `Map` автоматически управляет регионом, но можно задать `MapCameraPosition.rect`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKMapRect — bounding box маршрута с отступами 80 pt для комфортного отображения»

**Короткий итог 2026**:
> `MKMapRect` — это **прямоугольник** на карте в метрах (Mercator-проекция), используемый для bounding box и видимой области.  
> В 2026 году:  
> - создаётся через `MKMapRect(origin:size:)` или `MKMapRect(annotations:)` / `route.boundingMapRect`  
> - задаётся через `mapView.setVisibleMapRect(_:edgePadding:animated:)`  
> - самый удобный способ центрировать карту под маршрут, аннотации или оверлеи  
> Это **низкоуровневый**, но **очень точный** инструмент для работы с видимой областью карты.

Удачи с идеальным позиционированием и центрированием карты в твоём приложении! 🗺️📐