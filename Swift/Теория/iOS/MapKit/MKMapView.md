**`MKMapView`** — это основной класс в фреймворке **[[MapKit]]**, который отображает **интерактивную карту** (Apple Maps) в вашем [[iOS]]-приложении.

Это мощный и полностью нативный компонент для показа карт, маршрутов, аннотаций (pins), оверлеев, 3D-вида, Flyover и многого другого.

### Ключевые характеристики MKMapView (актуально на 2026 год)

| Характеристика                      | Описание                                                                    | Важные замечания 2026                          |
| ----------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------- |
| **Фреймворк**                       | MapKit                                                                      | Требует `import MapKit`                        |
| **Наследование**                    | `MKMapView` : [[UIView]]                                                    | Полностью нативный [[UIView]]                  |
| **Разрешения**                      | Требует `NSLocationWhenInUseUsageDescription` в Info.plist                  | Строгое поведение в iOS 18+ (Privacy Manifest) |
| **Типы карт**                       | `.standard`, `.satellite`, `.hybrid`, `.satelliteFlyover`, `.hybridFlyover` | Flyover — 3D-режим                             |
| **Аннотации**                       | Pins, кастомные аннотации (`MKAnnotationView`)                              | Поддержка кластеризации                        |
| **Оверлеи**                         | Полилинии, полигоны, круги (`MKOverlay`)                                    | Маршруты, зоны, границы                        |
| **Регионы**                         | `MKCoordinateRegion`, `MKMapRect`                                           | Управление видимой областью                    |
| **Пользовательское местоположение** | `showsUserLocation = true` + `userTrackingMode`                             | Синяя точка + heading                          |
| **Потокобезопасность**              | Делегат и обновления — **только главный поток**                             | [[@MainActor]] в Swift 6                       |

### Минимальный рабочий пример (современный стиль 2026)

```swift
import UIKit
import MapKit

@MainActor
final class MapViewController: UIViewController, MKMapViewDelegate, CLLocationManagerDelegate {
    
    private let mapView = MKMapView()
    private let locationManager = CLLocationManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupMapView()
        setupLocationManager()
    }
    
    private func setupMapView() {
        mapView.delegate = self
        mapView.showsUserLocation = true
        mapView.userTrackingMode = .followWithHeading  // Следить за пользователем + направление
        mapView.mapType = .hybridFlyover               // Гибрид + 3D Flyover
        mapView.preferredConfiguration = MKHybridMapConfiguration() // для Flyover
        
        view.addSubview(mapView)
        mapView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            mapView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            mapView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            mapView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            mapView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
    
    private func setupLocationManager() {
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        
        switch locationManager.authorizationStatus {
        case .authorizedWhenInUse, .authorizedAlways:
            mapView.showsUserLocation = true
            mapView.userTrackingMode = .follow
        case .notDetermined:
            locationManager.requestWhenInUseAuthorization()
        default:
            break
        }
    }
    
    // Делегат: статус геолокации изменился
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        if manager.authorizationStatus == .authorizedWhenInUse ||
           manager.authorizationStatus == .authorizedAlways {
            mapView.showsUserLocation = true
            mapView.userTrackingMode = .follow
        }
    }
    
    // Делегат: новая локация пользователя
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        // Центрируем карту на пользователе (если нужно)
        let region = MKCoordinateRegion(center: location.coordinate,
                                       latitudinalMeters: 1000,
                                       longitudinalMeters: 1000)
        mapView.setRegion(region, animated: true)
    }
    
    // Делегат: кастомизация аннотации (pin)
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
    
    // Делегат: пользователь тапнул на callout accessory
    func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
        if let annotation = view.annotation {
            print("Тап по аннотации:", annotation.title ?? "Без названия")
        }
    }
}
```

### Как добавить аннотацию (pin) на карту

```swift
let coordinate = CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194) // San Francisco
let annotation = MKPointAnnotation()
annotation.coordinate = coordinate
annotation.title = "San Francisco"
annotation.subtitle = "Город у залива"
mapView.addAnnotation(annotation)
```

### Лучшие практики [[MKMapViewDelegate]] в 2026

- **Делайте делегат отдельным классом** ([[NSObject]] + `@MainActor`)  
- **Проверяйте** `authorizationStatus` **перед** `showsUserLocation = true`  
- **Используйте** `MKMapCamera` для красивого 3D-вида и Flyover  
- **Для кастомных пинов** — реализуйте `viewFor annotation:` и используйте `MKMarkerAnnotationView` / `MKAnnotationView`  
- **Для маршрутов** — добавляйте `MKPolyline` через `addOverlay` и `rendererFor overlay`  
- **В SwiftUI** — оборачивайте в `UIViewRepresentable` + `Coordinator` (делегат)  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKMapViewDelegate — обработка аннотаций и пользовательской локации»

**Короткий итог 2026**:
> `MKMapViewDelegate` — это **мозг** карты: аннотации, оверлеи, пользовательская локация, зум и взаимодействие.  
> В 2026 году:  
> - главный метод — `mapView(_:viewFor:)` для кастомных пинов  
> - ключевой для локации — `didUpdateLocations`  
> - делегат — `@MainActor` + отдельный класс  
> - для SwiftUI — `UIViewRepresentable` + `Coordinator`  
> Это **основа** всех карт и навигации в iOS-приложениях.
