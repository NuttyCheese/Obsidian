**`CLLocationCoordinate2D`** — это простая структура в фреймворке **[[Core Location]]**, которая представляет географическую координату на поверхности Земли.

Это **самый часто используемый** тип для работы с местоположением в [[iOS]]-приложениях.

### Основные свойства структуры

```swift
public struct CLLocationCoordinate2D {
    public var latitude:  CLLocationDegrees   // широта  (-90.0 … +90.0)
    public var longitude: CLLocationDegrees   // долгота (-180.0 … +180.0)
}
```

- `latitude` — широта в градусах (от -90° юг до +90° север)
- `longitude` — долгота в градусах (от -180° запад до +180° восток)

### Самые популярные способы создания и использования (2026 стандарт)

#### 1. Простое создание координаты

```swift
let moscow = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)   // Красная площадь
let newYork = CLLocationCoordinate2D(latitude: 40.7128, longitude: -74.0060) // Манхэттен
let sydney  = CLLocationCoordinate2D(latitude: -33.8688, longitude: 151.2093)
```

#### 2. Получение текущих координат пользователя (самый частый сценарий)

```swift
import CoreLocation

let locationManager = CLLocationManager()

// После получения разрешения и startUpdatingLocation()
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.last else { return }
    
    let coordinate = location.coordinate  // ← здесь CLLocationCoordinate2D
    print("Текущие координаты: \(coordinate.latitude), \(coordinate.longitude)")
}
```

#### 3. Использование в [[MapKit]] ([[MKMapView]], аннотации, регионы)

```swift
// Центрирование карты
let center = CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194)
let region = MKCoordinateRegion(center: center,
                               latitudinalMeters: 2000,
                               longitudinalMeters: 2000)
mapView.setRegion(region, animated: true)

// Добавление аннотации
let annotation = MKPointAnnotation()
annotation.coordinate = center
annotation.title = "San Francisco"
mapView.addAnnotation(annotation)

// Геозона
let geofence = CLCircularRegion(center: center, radius: 500, identifier: "sf-zone")
locationManager.startMonitoring(for: geofence)
```

#### 4. Расчёт расстояния между двумя точками

```swift
let coord1 = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
let coord2 = CLLocationCoordinate2D(latitude: 59.9343, longitude: 30.3351)

let location1 = CLLocation(latitude: coord1.latitude, longitude: coord1.longitude)
let location2 = CLLocation(latitude: coord2.latitude, longitude: coord2.longitude)

let distance = location1.distance(from: location2)  // в метрах
print("Расстояние Москва ↔ Санкт-Петербург: \(distance / 1000) км")
```

### Лучшие практики CLLocationCoordinate2D в Swift 2026

- **Всегда** проверяйте валидность координат перед использованием  
  ```swift
  func isValidCoordinate(_ coord: CLLocationCoordinate2D) -> Bool {
      return coord.latitude >= -90 && coord.latitude <= 90 &&
             coord.longitude >= -180 && coord.longitude <= 180
  }
  ```

- **Храните** координаты как `CLLocationCoordinate2D`, а не как отдельные [[Double]] — это типобезопаснее  
- **Для отображения** — используйте [[CLLocationFormatter]] или [[CLGeocoder]] для читаемого адреса  
- **Для сравнения** — используйте `CLLocation.distance(from:)` вместо ручного расчёта  
- **В SwiftUI** — передавайте `CLLocationCoordinate2D` в `Map` / `MapMarker` напрямую  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании Core Location  
- **Документируйте** — пишите комментарий «CLLocationCoordinate2D — координаты центра Москвы (Красная площадь)»

**Короткий итог 2026**:
> `CLLocationCoordinate2D` — это **базовая структура** для представления любой точки на Земле (широта + долгота).  
> В 2026 году:  
> - используется **везде**, где нужна геолокация: карты, геозоны, маршруты, аннотации  
> - создаётся напрямую или берётся из `CLLocation.coordinate`  
> - ключевые методы: `MKMapView.setRegion`, `CLCircularRegion`, `CLLocation.distance(from:)`  
> Это **самый фундаментальный** тип для всей геолокационной работы в iOS-приложениях.
