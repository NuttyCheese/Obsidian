**`MKOverlay`** — это **протокол** в фреймворке **[[MapKit]]**, который определяет объект как **оверлей** (overlay) на карте [[MKMapView]].

Оверлей — это **графический слой поверх карты**, который может представлять:
- линии (маршруты, дороги, границы) — [[MKPolyline]]
- полигоны (зоны, страны, здания) — [[MKPolygon]]
- круги (радиус покрытия, геозоны) — [[MKCircle]]
- кастомные оверлеи (любые сложные формы)

Сам протокол `MKOverlay` наследуется от [[MKAnnotation]] и добавляет одно ключевое свойство:

```swift
public protocol MKOverlay : MKAnnotation {
    var coordinate: CLLocationCoordinate2D { get }
    var boundingMapRect: MKMapRect { get }   // ← обязательное
}
```

`boundingMapRect` — это прямоугольная область (в проекции карты), в которой находится весь оверлей. MapKit использует её для оптимизации рендеринга.

### Основные встроенные классы, реализующие MKOverlay

| Класс                | Что рисует                                | Самый частый сценарий использования      | Как создать                              |
| -------------------- | ----------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| `MKPolyline`         | Ломаная линия / маршрут                   | Отображение маршрутов ([[MKDirections]]) | `MKPolyline(points:coordinates, count:)` |
| `MKPolygon`          | Замкнутый полигон (с дырками опционально) | Зоны, границы, страны, здания            | `MKPolygon(coordinates:count:)`          |
| `MKCircle`           | Круг с заданным центром и радиусом        | Радиус доставки, geofencing              | `MKCircle(center:radius:)`               |
| `MKMultiPoint`       | Набор точек (редко используется напрямую) | —                                        | —                                        |
| `MKGeodesicPolyline` | Геодезическая (кратчайшая по сфере) линия | Длинные маршруты по глобусу              | `MKGeodesicPolyline(points:count:)`      |

### Как добавить оверлей на карту (обязательный делегат)

```swift
class MapViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        
        // Пример: круг радиусом 2 км вокруг центра Москвы
        let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
        let circle = MKCircle(center: center, radius: 2000)
        circle.title = "Зона покрытия"
        circle.subtitle = "2 км"
        
        mapView.addOverlay(circle)
        
        // Центрируем карту
        let region = MKCoordinateRegion(center: center,
                                       latitudinalMeters: 5000,
                                       longitudinalMeters: 5000)
        mapView.setRegion(region, animated: true)
    }
    
    // Обязательный метод делегата — рендеринг оверлея
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        if let circle = overlay as? MKCircle {
            let renderer = MKCircleRenderer(circle: circle)
            renderer.fillColor = UIColor.systemBlue.withAlphaComponent(0.25)
            renderer.strokeColor = UIColor.systemBlue
            renderer.lineWidth = 3
            renderer.alpha = 0.8
            
            return renderer
        }
        
        // Для полилинии (маршрута)
        if let polyline = overlay as? MKPolyline {
            let renderer = MKPolylineRenderer(polyline: polyline)
            renderer.strokeColor = .systemGreen
            renderer.lineWidth = 6
            renderer.lineCap = .round
            renderer.lineJoin = .round
            
            return renderer
        }
        
        return MKOverlayRenderer(overlay: overlay)
    }
}
```

### Полный пример с маршрутом (MKDirections + MKPolyline)

```swift
func showRoute(from start: CLLocationCoordinate2D, to end: CLLocationCoordinate2D) {
    let request = MKDirections.Request()
    request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
    request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
    request.transportType = .automobile
    
    let directions = MKDirections(request: request)
    directions.calculate { response, error in
        guard let route = response?.routes.first else { return }
        
        // Добавляем маршрут как оверлей
        self.mapView.addOverlay(route.polyline)
        
        // Центрируем карту на весь маршрут
        self.mapView.setVisibleMapRect(route.polyline.boundingMapRect,
                                      edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                                      animated: true)
    }
}
```

### Лучшие практики [[MKOverlay]] / [[MKOverlayRenderer]] в Swift 2026

- **Всегда** реализуйте `mapView(_:rendererFor:)` — без него оверлей не отобразится  
- **Используйте** конкретные рендереры: `MKPolylineRenderer`, `MKCircleRenderer`, `MKPolygonRenderer`  
- **Для маршрутов** — добавляйте `MKPolyline` из `MKRoute.polyline`  
- **Для динамических оверлеев** — обновляйте через `removeOverlay` + `addOverlay`  
- **Для производительности** — не добавляйте слишком много оверлеев (MapKit сам оптимизирует)  
- **В [[SwiftUI]]** — оборачивайте `MKMapView` в [[UIViewRepresentable]] и добавляйте оверлеи в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKOverlay — круг радиусом 2 км с полупрозрачной заливкой (MKCircleRenderer)»

**Короткий итог 2026**:
> `MKOverlay` — это протокол для **графических слоёв поверх карты** (линии, круги, полигоны).  
> В 2026 году:  
> - основные классы — `MKCircle`, `MKPolyline`, `MKPolygon`  
> - добавляются через `mapView.addOverlay`  
> - рендерятся в `mapView(_:rendererFor:)` с помощью `MK*Renderer`  
> - это **единственный** способ отобразить маршруты, зоны покрытия, границы и другие векторные фигуры на карте  
