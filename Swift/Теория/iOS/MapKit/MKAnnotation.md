**`MKAnnotation`** — это **протокол** в фреймворке **[[MapKit]]**, который определяет объект как **аннотацию** (annotation) на карте [[MKMapView]].

Аннотация — это **маркер** (pin, метка, точка интереса), который отображается на карте в определённой географической координате и может иметь заголовок, подзаголовок, кастомный вид и callout (всплывающее окно при тапе).

### Основные требования протокола MKAnnotation

```swift
public protocol MKAnnotation : NSObjectProtocol {
    var coordinate: CLLocationCoordinate2D { get set }   // ← обязательное
    var title: String? { get }                           // опционально
    var subtitle: String? { get }                        // опционально
}
```

- `coordinate` — единственное **обязательное** свойство
- `title` и `subtitle` — опциональные, отображаются в callout
- Можно добавить любые другие свойства (например, `image`, `identifier`, `data`)

### Самые популярные способы реализации MKAnnotation в 2026 году

#### 1. Простейшая реализация (самый частый случай)

```swift
class PlaceAnnotation: NSObject, MKAnnotation {
    let coordinate: CLLocationCoordinate2D
    let title: String?
    let subtitle: String?
    
    init(coordinate: CLLocationCoordinate2D, title: String? = nil, subtitle: String? = nil) {
        self.coordinate = coordinate
        self.title = title
        self.subtitle = subtitle
        super.init()
    }
}
```

Использование:

```swift
let annotation = PlaceAnnotation(
    coordinate: CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173),
    title: "Красная площадь",
    subtitle: "Москва, Россия"
)

mapView.addAnnotation(annotation)
```

#### 2. Более практичная модель (с дополнительными данными)

```swift
class RestaurantAnnotation: NSObject, MKAnnotation {
    let coordinate: CLLocationCoordinate2D
    let restaurant: Restaurant
    
    var title: String? { restaurant.name }
    var subtitle: String? { restaurant.cuisine }
    
    init(restaurant: Restaurant) {
        self.restaurant = restaurant
        self.coordinate = restaurant.coordinate
        super.init()
    }
}

struct Restaurant {
    let name: String
    let cuisine: String
    let coordinate: CLLocationCoordinate2D
}
```

#### 3. Кастомная аннотация с изображением (очень популярно)

```swift
class CustomPinAnnotation: NSObject, MKAnnotation {
    let coordinate: CLLocationCoordinate2D
    let identifier: String
    let image: UIImage?
    
    var title: String? { "Custom Pin" }
    
    init(coordinate: CLLocationCoordinate2D, identifier: String, image: UIImage? = nil) {
        self.coordinate = coordinate
        self.identifier = identifier
        self.image = image
        super.init()
    }
}
```

### Как отобразить кастомную аннотацию (самый важный делегат)

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if annotation is MKUserLocation { return nil } // стандартная синяя точка
    
    let identifier = "CustomPin"
    var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
    
    if annotationView == nil {
        annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        annotationView?.canShowCallout = true
        annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
        
        // Кастомное изображение
        if let customAnnotation = annotation as? CustomPinAnnotation,
           let image = customAnnotation.image {
            (annotationView as? MKMarkerAnnotationView)?.glyphImage = image
        }
    } else {
        annotationView?.annotation = annotation
    }
    
    return annotationView
}
```

### Лучшие практики MKAnnotation в Swift 2026

- **Делайте** класс, а не struct — протокол требует `NSObjectProtocol`  
- **Храните** `id` / `identifier` — очень полезно для `dequeueReusableAnnotationView`  
- **Используйте** `MKMarkerAnnotationView` (iOS 11+) — современный стиль пинов с цветом и glyph  
- **Для кластеризации** — реализуйте `MKAnnotation` + [[MKClusterAnnotation]] (автоматическая группировка)  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и используйте `mapView(_:viewFor:)` в `Coordinator`  
- **Документируйте** — пишите комментарий «MKAnnotation — аннотация для ресторана с кастомным изображением»

**Короткий итог 2026**:
> `MKAnnotation` — это протокол, который превращает любой объект в **маркер на карте**.  
> В 2026 году:  
> - минимум — `coordinate` (обязательно) + `title` / `subtitle` (опционально)  
> - чаще всего реализуют как класс с `NSObject`  
> - для кастомного вида — [[MKMarkerAnnotationView]] + `glyphImage` / `glyphText`  
> - ключевой делегат — `mapView(_:viewFor:)`  
> Это **основа** всех маркеров, пинов и точек интереса на картах в iOS-приложениях.
