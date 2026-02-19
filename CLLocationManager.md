**`CLLocationManager`** — это основной класс в **Core Location** фреймворке для работы с геолокацией на iOS, iPadOS, macOS, watchOS и tvOS.

Он отвечает за:
- получение текущего местоположения устройства,
- отслеживание изменений местоположения,
- работу с регионами (geofencing),
- получение заголовков (heading),
- мониторинг beacons,
- запрос разрешений на геолокацию.

### Ключевые особенности CLLocationManager (актуально на 2026 год)

| Характеристика                     | Описание                                                                 | Важные замечания 2026 |
|------------------------------------|--------------------------------------------------------------------------|------------------------|
| **Разрешения**                     | Требует запроса `requestWhenInUseAuthorization()` или `requestAlwaysAuthorization()` | Строгое поведение в iOS 17+ (Privacy Manifest, NSLocation*) |
| **Точность**                       | `desiredAccuracy`: `kCLLocationAccuracyBest`, `kCLLocationAccuracyKilometer` и др. | `bestForNavigation` — самый точный, но энергозатратный |
| **Фоновый режим**                  | Поддерживается при `always` разрешении + Background Modes → Location updates | С iOS 18+ ещё строже (App Privacy Report) |
| **Significant-change location**    | `startMonitoringSignificantLocationChanges()` — экономит батарею           | Работает даже при killed-приложении |
| **Region monitoring**              | Геозоны (circular regions) до 20 на приложение                           | iBeacon + CLBeaconRegion |
| **Heading**                        | `startUpdatingHeading()` — компас (магнитный/истинный)                   | Требует отдельного разрешения (NSLocationWhenInUseUsageDescription) |
| **Потокобезопасность**             | Делегат вызывается на **главном потоке**                                 | С Swift 6 — рекомендуется `@MainActor` |

### Минимальный рабочий пример (2026 стандарт)

```swift
import CoreLocation

@MainActor
class LocationService: NSObject, CLLocationManagerDelegate {
    private let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = 10 // метров
        locationManager.pausesLocationUpdatesAutomatically = true
    }
    
    func requestLocation() {
        // Проверка статуса
        switch locationManager.authorizationStatus {
        case .notDetermined:
            locationManager.requestWhenInUseAuthorization() // или requestAlways
        case .authorizedWhenInUse, .authorizedAlways:
            locationManager.startUpdatingLocation()
        case .restricted, .denied:
            print("Геолокация запрещена")
        @unknown default:
            break
        }
    }
    
    // Делегат: новое местоположение
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        print("Текущее местоположение: \(location.coordinate.latitude), \(location.coordinate.longitude)")
        print("Точность: \(location.horizontalAccuracy) м")
        
        // Можно остановить обновления, если не нужны постоянно
        // manager.stopUpdatingLocation()
    }
    
    // Делегат: ошибка
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Ошибка геолокации:", error.localizedDescription)
    }
    
    // Делегат: изменение разрешения
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        if manager.authorizationStatus == .authorizedWhenInUse ||
           manager.authorizationStatus == .authorizedAlways {
            manager.startUpdatingLocation()
        }
    }
}
```

### Ключевые методы и свойства CLLocationManager

| Метод / Свойство                        | Что делает                                               | Важные замечания 2026 |
|-----------------------------------------|----------------------------------------------------------|------------------------|
| `requestWhenInUseAuthorization()`       | Запрос "при использовании"                               | Самый частый выбор     |
| `requestAlwaysAuthorization()`          | Запрос "всегда" (фоновый режим)                          | Требует Privacy Manifest |
| `startUpdatingLocation()`               | Начать получать обновления местоположения                | Вызывать после разрешения |
| `stopUpdatingLocation()`                | Остановить обновления                                    | Экономия батареи       |
| `startMonitoringSignificantLocationChanges()` | Экономичный фоновый режим (при смене сотовой вышки) | Работает при killed-приложении |
| `authorizationStatus`                   | Текущий статус разрешения                                | `.authorizedAlways`, `.authorizedWhenInUse` и др. |
| `accuracyAuthorization`                 | `.fullAccuracy` / `.reducedAccuracy` (iOS 14+)           | Пользователь может ограничить точность |
| `desiredAccuracy`                       | Уровень точности (best, kilometer, nearestTenMeters и др.) | `bestForNavigation` — самый точный |

### Лучшие практики CLLocationManager в 2026

- **Всегда** делайте класс-обёртку (`LocationService`, `LocationManagerWrapper`) и используйте **делегат** как `NSObject` + `@MainActor`  
- **Проверяйте** `authorizationStatus` **перед** запуском обновлений  
- **Останавливайте** обновления (`stopUpdatingLocation()`) когда они не нужны — это критично для батареи  
- **Используйте** `CLLocationManager` как **singleton** или **shared instance** в большинстве приложений  
- **Добавляйте** в Info.plist:
  - `NSLocationWhenInUseUsageDescription`
  - `NSLocationAlwaysAndWhenInUseUsageDescription` (если нужен always)
  - `NSLocationAlwaysUsageDescription` (устарело, но иногда нужно)
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для геолокации  
- **В SwiftUI** — чаще используют `.task { await requestLocation() }` + `@State` / `@ObservedObject`  
- **Документируйте** — пишите комментарий «CLLocationManager — получение геолокации с обработкой разрешений»

**Короткий итог 2026**:
> `CLLocationManager` — центральный класс для всей геолокации в iOS.  
> В 2026 году:  
> - запрашивайте разрешение **до** старта обновлений  
> - используйте `authorizationStatus` и `accuracyAuthorization`  
> - останавливайте обновления, когда не нужны  
> - всегда оборачивайте в отдельный класс с делегатом  
> - учитывайте Privacy Manifest и фоновые режимы  

Удачи с корректной и энергосберегающей геолокацией в твоём приложении! 📍