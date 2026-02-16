**Core Location** — это основной фреймворк Apple для работы с **геолокацией**, **движением**, **компасом**, **iBeacon** и **региональным мониторингом** на всех платформах Apple (iOS, iPadOS, macOS, watchOS, tvOS, visionOS).

По состоянию на февраль 2026 года Core Location остаётся **единственным официальным способом** получения точных геоданных и событий движения на устройствах Apple.

### Ключевые возможности Core Location (актуальные 2026)

| Возможность                          | Класс / API                              | Минимальная версия ОС | Точность / энергопотребление | Когда использовать в 2026 |
|--------------------------------------|------------------------------------------|------------------------|-------------------------------|----------------------------|
| **Текущее местоположение**           | `CLLocationManager` + `requestLocation()` | iOS 9+                 | Высокая (GPS + Wi-Fi + Cell)  | Основной способ получения координат |
| **Значимые изменения местоположения** | `startMonitoringSignificantLocationChanges()` | iOS 4+                 | Низкое энергопотребление      | Фоновый трекинг (например, погода по городу) |
| **Региональный мониторинг**          | `CLCircularRegion` + `startMonitoring(for:)` | iOS 4+                 | Среднее                       | Геозоны (вход/выход из региона) |
| **iBeacon (Bluetooth beacons)**      | `CLBeaconRegion` + `startMonitoring(for:)` | iOS 7+                 | Очень низкое энергопотребление | Магазины, музеи, indoor-навигация |
| **Heading (компас)**                 | `startUpdatingHeading()`                 | iOS 3+                 | Среднее                       | AR, карты, навигация |
| **Motion Activity**                  | `CMMotionActivityManager` (Core Motion) + Core Location | iOS 7+                 | Низкое                        | Определение ходьбы/бега/транспорта |
| **Deferred location updates**        | `allowDeferredLocationUpdates(untilTraveled:timeout:)` | iOS 6+                 | Очень низкое                  | Долгие поездки (экономия батареи) |
| **Authorization**                    | `requestWhenInUseAuthorization()` / `requestAlwaysAuthorization()` | iOS 8+                 | —                             | Обязательно перед использованием |
| **CLLocationButton** (iOS 15+)       | `CLLocationButton` (SwiftUI)             | iOS 15+                | —                             | Простой доступ к локации из UI |
| **Live Location** (iOS 18+)          | Поддержка в Find My / Messages           | iOS 18+                | —                             | Шеринг локации в реальном времени |

### Самый популярный и рекомендуемый паттерн 2026 года

#### Современный подход: CLLocationManager + async/await + actor

```swift
import CoreLocation

actor LocationService: NSObject, CLLocationManagerDelegate {
    
    private let locationManager = CLLocationManager()
    private var continuation: CheckedContinuation<CLLocation, Error>?
    
    override init() {
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = 10
        locationManager.activityType = .fitness  // или .automotiveNavigation и т.д.
    }
    
    func requestLocation() async throws -> CLLocation {
        switch locationManager.authorizationStatus {
        case .notDetermined:
            locationManager.requestWhenInUseAuthorization()
            // Ждём разрешения (можно добавить timeout)
            try await Task.sleep(for: .seconds(5))
            
        case .authorizedWhenInUse, .authorizedAlways:
            break
            
        default:
            throw LocationError.denied
        }
        
        return try await withCheckedThrowingContinuation { continuation in
            self.continuation = continuation
            locationManager.requestLocation()  // Один запрос
        }
    }
    
    // Делегат
    nonisolated func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        
        Task { @MainActor in
            continuation?.resume(returning: location)
            continuation = nil
        }
    }
    
    nonisolated func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        Task { @MainActor in
            continuation?.resume(throwing: error)
            continuation = nil
        }
    }
    
    nonisolated func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        // Можно добавить логику при смене авторизации
    }
}

// Использование
@MainActor
class LocationViewModel: ObservableObject {
    @Published var location: CLLocation?
    @Published var error: Error?
    
    private let service = LocationService()
    
    func fetchLocation() async {
        do {
            location = try await service.requestLocation()
        } catch {
            self.error = error
        }
    }
}
```

### Лучшие практики Core Location в Swift 2026

- **Всегда используй actor** для хранения `CLLocationManager` — это предотвращает data race  
- **requestLocation()** — вместо `startUpdatingLocation()` в большинстве случаев (один точный запрос)  
- **requestWhenInUseAuthorization()** — основной вариант (если не нужен фон)  
- **requestAlwaysAuthorization()** — только если реально нужен фон (Apple очень строго проверяет)  
- **CLLocationManagerDelegate** — реализуй как `nonisolated` методы, вызывай continuation в `@MainActor`  
- **DesiredAccuracy** — используй `kCLLocationAccuracyBestForNavigation` для навигации, `kCLLocationAccuracyHundredMeters` для экономии  
- **Significant-change location** — для экономии батареи в фоне  
- **Swift 6 strict concurrency** — весь UI и делегат — `@MainActor`, менеджер — в `actor`  
- **Тестирование** — используй `CLLocationManager` mock / stub (ручной или библиотеки типа Cuckoo)  
- **Документируйте** — пишите комментарий «Core Location — получение геопозиции с учётом авторизации»

**Короткий девиз 2026**:
> «Core Location в 2026 году — это когда тебе нужно точное местоположение пользователя.  
> Самый современный стиль — actor + async/await + requestLocation().  
> Избегай startUpdatingLocation() в большинстве случаев — это энергозатратно и устарело.»

Удачи с точной и энергоэффективной геолокацией в Swift! 📍