**`MKMapPoint`** — это структура в фреймворке **MapKit**, которая представляет **точку на карте в проекции Mercator** (2D-плоская проекция, используемая Apple Maps).

Это **низкоуровневый** тип координат, который MapKit использует **внутри** для всех расчётов, bounding box'ов, оверлеев и аннотаций.  
В отличие от [[CLLocationCoordinate2D]] (широта/долгота в градусах), `MKMapPoint` работает в **метрах** относительно центра проекции (0,0 — точка на экваторе на нулевом меридиане).

### Основные свойства структуры MKMapPoint

```swift
public struct MKMapPoint {
    public var x: Double     // горизонтальная координата (восток-запад), метры
    public var y: Double     // вертикальная координата (север-юг), метры
}
```

- `x` — расстояние по восток-запад (longitude в метрах)
- `y` — расстояние по север-юг (latitude в метрах)

### Самые важные и часто используемые преобразования (2026 стандарт)

#### 1. Из CLLocationCoordinate2D в MKMapPoint (самый частый)

```swift
let coordinate = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва

let mapPoint = MKMapPoint(coordinate)  // ← преобразование

print("x: \(mapPoint.x), y: \(mapPoint.y)")
```

#### 2. Из MKMapPoint обратно в CLLocationCoordinate2D

```swift
let mapPoint = MKMapPoint(x: 12345678.9, y: 9876543.2)

let coordinate = mapPoint.coordinate  // ← обратное преобразование
print("Широта: \(coordinate.latitude), Долгота: \(coordinate.longitude)")
```

#### 3. Расстояние между двумя MKMapPoint (в метрах)

```swift
let p1 = MKMapPoint(coordinate1)
let p2 = MKMapPoint(coordinate2)

let distance = p1.distance(to: p2)  // в метрах
print("Расстояние: \(distance / 1000) км")
```

#### 4. Bounding box для набора точек (часто для маршрутов/аннoтаций)

```swift
let points = [startPoint, endPoint]  // [MKMapPoint]

let rect = MKMapRect(points: points)  // или route.boundingMapRect

mapView.setVisibleMapRect(rect, 
                         edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                         animated: true)
```

### Когда и зачем использовать MKMapPoint вместо CLLocationCoordinate2D

| Ситуация                                              | Почему именно MKMapPoint                                  | Альтернатива (когда можно обойтись) |
| ----------------------------------------------------- | --------------------------------------------------------- | ----------------------------------- |
| Расчёт точного расстояния между точками               | `distance(to:)` работает в метрах, без искажений проекции | `CLLocation.distance(from:)`        |
| Работа с bounding box / visibleMapRect                | `MKMapRect` использует именно MKMapPoint                  | `MKCoordinateRegion`                |
| Оверлеи ([[MKPolyline]], [[MKPolygon]], [[MKCircle]]) | `points` и `boundingMapRect` — это MKMapPoint             | —                                   |
| Кластеризация аннотаций                               | Внутренние расчёты MapKit используют MKMapPoint           | —                                   |
| Оптимизация производительности                        | Избегает повторных преобразований координат               | `CLLocationCoordinate2D`            |

### Лучшие практики MKMapPoint в Swift 2026

- **Используйте** `MKMapPoint(coordinate:)` только когда действительно нужен расчёт в метрах или работа с `MKMapRect`  
- **Для обычных задач** (аннотации, регионы, геозоны) — работайте с `CLLocationCoordinate2D`  
- **Для маршрутов** — берите `route.boundingMapRect` — это уже в MKMapPoint  
- **Для расстояний** — предпочтительнее `CLLocation.distance(from:)` — проще и точнее  
- **В SwiftUI** — `Map` работает с `CLLocationCoordinate2D`, преобразование в MKMapPoint происходит автоматически  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании MapKit  
- **Документируйте** — пишите комментарий «MKMapPoint — проекция координат для расчёта boundingMapRect маршрута»

**Короткий итог 2026**:
> `MKMapPoint` — это **плоская проекция** координат в метрах (Mercator), используемая MapKit для внутренних расчётов.  
> В 2026 году:  
> - получайте через `MKMapPoint(coordinate:)`  
> - используйте для: `distance(to:)`, `MKMapRect`, оверлеев, bounding box  
> - для повседневных задач (аннотации, регионы) — почти всегда достаточно `CLLocationCoordinate2D`  
> Это **низкоуровневый** тип, который нужен, когда требуется точный расчёт в метрах или работа с прямоугольниками карты.

Удачи с точными расчётами и эффективным позиционированием карты в твоём проекте! 🗺️📐