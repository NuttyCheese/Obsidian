**`CLLocationManagerDelegate`** — это **протокол** в фреймворке **Core Location**, который определяет методы-делегаты для получения событий и обновлений от объекта **`CLLocationManager`**.

Это **единственный** способ получать геолокацию, ошибки, изменения авторизации и другие события от менеджера местоположения.

### Основные методы протокола CLLocationManagerDelegate (актуально на 2026 год)

| Метод делегата                                      | Когда вызывается                                                                 | Самый частый сценарий использования в 2026 |
|-----------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------|
| `locationManager(_:didUpdateLocations:)`            | Пришло новое местоположение (или несколько)                                      | Основной метод — получение координат       |
| `locationManager(_:didFailWithError:)`              | Ошибка получения местоположения (нет GPS, запрет, timeout и т.д.)                | Обработка ошибок, показ пользователю       |
| `locationManagerDidChangeAuthorization(_:)`         | Изменился статус авторизации (пользователь дал/отозвал разрешение)               | Самый важный метод в iOS 14+               |
| `locationManager(_:didUpdateHeading:)`              | Обновился курс/направление (heading)                                             | Компас, AR-навигация                       |
| `locationManager(_:didEnterRegion:)`                | Устройство вошло в регион (геозона)                                              | Geofencing, уведомления при входе          |
| `locationManager(_:didExitRegion:)`                 | Устройство вышло из региона                                                      | Geofencing                                 |
| `locationManager(_:didDetermineState:for:)`         | Определено текущее состояние региона (inside/outside/unknown)                    | Geofencing                                 |
| `locationManager(_:monitoringDidFailFor:withError:)`| Ошибка мониторинга региона                                                       | Обработка ошибок геозон                    |
| `locationManager(_:didRangeBeacons:in:)`            | Обновились данные по iBeacon (ranging)                                           | Proximity-уведомления, indoor-навигация    |
| `locationManager(_:didChangeDeviceOrientation:)`    | Изменилась ориентация устройства (редко используется)                            | —                                          |

### Самый важный метод в 2026 году

```swift
func locationManagerDidChangeAuthorization(_ manager: CLLocationManager)
```

Этот метод **заменил** устаревший `locationManager(_:didChangeAuthorizationStatus:)` (deprecated с iOS 14).

Почему он критически важен:
- вызывается **при первом запуске** и **при каждом изменении** статуса
- позволяет **запустить** `startUpdatingLocation()` сразу после получения разрешения
- учитывает **новые статусы** `.authorizedAlways`, `.authorizedWhenInUse`, `.restricted`, `.denied`

### Полный современный пример реализации делегата (2026 стандарт)

```swift
import CoreLocation

@MainActor
final class LocationService: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
        manager.distanceFilter = 10
        manager.pausesLocationUpdatesAutomatically = true
        manager.allowsBackgroundLocationUpdates = true // если нужен always
    }
    
    func checkAndRequestAuthorization() {
        switch manager.authorizationStatus {
        case .notDetermined:
            manager.requestWhenInUseAuthorization() // или requestAlways
        case .authorizedWhenInUse, .authorizedAlways:
            startUpdating()
        case .restricted, .denied:
            showLocationDisabledAlert()
        @unknown default:
            break
        }
    }
    
    private func startUpdating() {
        manager.startUpdatingLocation()
        // manager.startUpdatingHeading() — если нужен компас
        // manager.startMonitoringSignificantLocationChanges() — экономичный фон
    }
    
    // Самый важный метод
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        checkAndRequestAuthorization()
    }
    
    // Основной метод получения координат
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        let coordinate = location.coordinate
        print("Широта: \(coordinate.latitude), Долгота: \(coordinate.longitude)")
        print("Точность: ±\(location.horizontalAccuracy) м")
        print("Высота: \(location.altitude) м")
        
        // Можно остановить, если точность достаточна
        if location.horizontalAccuracy < 20 {
            manager.stopUpdatingLocation()
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Ошибка геолокации:", error.localizedDescription)
        
        if (error as NSError).code == CLError.denied.rawValue {
            showLocationDisabledAlert()
        }
    }
    
    private func showLocationDisabledAlert() {
        // Показать алерт с переходом в настройки
    }
}
```

### Лучшие практики CLLocationManagerDelegate в 2026

- **Делайте делегат отдельным классом** (`NSObject` + `@MainActor`)  
- **Проверяйте** `authorizationStatus` **в** `locationManagerDidChangeAuthorization` и **перед** `startUpdatingLocation()`  
- **Останавливайте** обновления (`stopUpdatingLocation()`) как можно раньше — это критично для батареи  
- **Обрабатывайте** `didFailWithError` — особенно `CLError.denied`, `CLError.locationUnknown`  
- **Для фонового режима** — используйте `startMonitoringSignificantLocationChanges()` + `always` разрешение  
- **В SwiftUI** — оборачивайте в `@ObservableObject` / `@StateObject` и используйте `.task { await checkAuthorization() }`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для всех методов Core Location  
- **Документируйте** — пишите комментарий «locationManager(_:didUpdateLocations:) — обработка новых координат с точностью < 20 м»

**Короткий итог 2026**:
> `CLLocationManagerDelegate` — это **единственный** способ получать геоданные, ошибки и изменения разрешений от `CLLocationManager`.  
> В 2026 году:  
> - главный метод — `locationManagerDidChangeAuthorization`  
> - основной рабочий — `didUpdateLocations`  
> - всегда обрабатывайте `didFailWithError`  
> - делегат — `@MainActor` + отдельный класс  
> - учитывайте Privacy Manifest и фоновые режимы  

Удачи с надёжным и энергосберегающим получением геолокации в твоём приложении! 🌍