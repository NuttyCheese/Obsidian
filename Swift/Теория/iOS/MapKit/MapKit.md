**MapKit** — это мощный фреймворк от Apple для работы с картами, геолокацией и навигацией в [[iOS]], iPadOS, macOS, watchOS и tvOS.

Он включает в себя:
- отображение интерактивных карт (Apple Maps),
- аннотации (pins, маркеры),
- оверлеи (маршруты, полигоны, круги),
- поиск мест,
- построение маршрутов,
- 3D Flyover,
- поддержку пользовательского местоположения и heading (компас),
- работу с регионами (geofencing) и iBeacon.

### Основные классы MapKit (актуально на 2026 год)

| Класс / Протокол                                            | Назначение                                                | Самый частый сценарий использования |
| ----------------------------------------------------------- | --------------------------------------------------------- | ----------------------------------- |
| [[MKMapView]]                                               | Основной view для отображения карты                       | Показ карты в приложении            |
| [[MKMapViewDelegate]]                                       | Делегат для кастомизации аннотаций, оверлеев, зума и т.д. | Кастомные пины, маршруты, parallax  |
| [[CLLocationManager]]                                       | Получение геолокации и heading                            | Синяя точка пользователя, heading   |
| [[MKAnnotation]] / [[MKPointAnnotation]]                    | Протокол и базовый класс для аннотаций (pins)             | Маркеры на карте                    |
| [[MKAnnotationView]] / [[MKMarkerAnnotationView]]           | View для отображения аннотации                            | Кастомные пины, кластеризация       |
| [[MKOverlay]] / [[MKPolyline]], [[MKPolygon]], [[MKCircle]] | Оверлеи на карте (линии, полигоны, круги)                 | Маршруты, зоны, границы             |
| [[MKDirections]]                                            | Построение маршрутов (Directions [[API]])                 | Навигация от точки A до точки B     |
| [[MKLocalSearch]]                                           | Поиск мест (POI) по запросу                               | Поиск кафе, магазинов и т.д.        |
| [[MKMapCamera]]                                             | Камера карты (угол наклона, поворот, высота)              | 3D Flyover, кастомный зум           |

### Минимальный рабочий пример MKMapView (современный стиль 2026)

```swift
import UIKit
import MapKit

@MainActor
final class MapViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func loadView() {
        mapView.delegate = self
        mapView.showsUserLocation = true
        mapView.userTrackingMode = .followWithHeading  // Следить + направление
        mapView.mapType = .hybridFlyover               // Гибрид + 3D Flyover
        
        view = mapView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Центрируем на известной локации (например, Москва)
        let coordinate = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
        let region = MKCoordinateRegion(center: coordinate,
                                       latitudinalMeters: 5000,
                                       longitudinalMeters: 5000)
        mapView.setRegion(region, animated: true)
        
        // Добавляем аннотацию
        let annotation = MKPointAnnotation()
        annotation.coordinate = coordinate
        annotation.title = "Красная площадь"
        annotation.subtitle = "Москва, Россия"
        mapView.addAnnotation(annotation)
    }
    
    // Кастомизация пина
    func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
        if annotation is MKUserLocation { return nil } // стандартная синяя точка
        
        let identifier = "CustomPin"
        var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
        
        if annotationView == nil {
            annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
            annotationView?.canShowCallout = true
            annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
        } else {
            annotationView?.annotation = annotation
        }
        
        return annotationView
    }
    
    // Тап по кнопке в callout
    func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
        if let annotation = view.annotation {
            print("Открыть детали для:", annotation.title ?? "Без названия")
        }
    }
}
```

### Как добавить маршрут (полилинию) на карту

```swift
func showRoute(from start: CLLocationCoordinate2D, to end: CLLocationCoordinate2D) {
    let request = MKDirections.Request()
    request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
    request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
    request.transportType = .automobile
    
    let directions = MKDirections(request: request)
    directions.calculate { response, error in
        guard let route = response?.routes.first else {
            print("Ошибка построения маршрута:", error?.localizedDescription ?? "Неизвестно")
            return
        }
        
        self.mapView.addOverlay(route.polyline)
        self.mapView.setVisibleMapRect(route.polyline.boundingMapRect, edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50), animated: true)
    }
}

// Делегат для рендеринга оверлея
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let polyline = overlay as? MKPolyline {
        let renderer = MKPolylineRenderer(polyline: polyline)
        renderer.strokeColor = .systemBlue
        renderer.lineWidth = 6
        renderer.alpha = 0.8
        return renderer
    }
    return MKOverlayRenderer(overlay: overlay)
}
```

### Лучшие практики MKMapView в Swift 2026

- **Всегда** устанавливайте `delegate` и реализуйте `viewFor annotation:` для кастомных пинов  
- **Используйте** `MKMapCamera` для красивого 3D-вида и Flyover  
- **Для маршрутов** — добавляйте `MKPolyline` через `addOverlay` и `rendererFor overlay`  
- **Для пользовательской локации** — `showsUserLocation = true` + `userTrackingMode` + обработка через [[CLLocationManagerDelegate]]  
- **В [[SwiftUI]]** — оборачивайте в `UIViewRepresentable` + [[Coordinator]] (делегат)  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «[[MKMapViewDelegate]] — кастомизация аннотаций и маршрутов»

**Короткий итог 2026**:
> `MKMapView` — это **нативная и мощная** карта Apple внутри вашего приложения.  
> В 2026 году:  
> - главный делегат — `MKMapViewDelegate`  
> - ключевые методы — `viewFor annotation:`, `rendererFor overlay:`  
> - для маршрутов — `MKDirections` + `MKPolyline`  
> - для пользовательской локации — `showsUserLocation` + `CLLocationManager`  
> Это **единственный** рекомендуемый способ показывать карты и навигацию в iOS-приложениях.
