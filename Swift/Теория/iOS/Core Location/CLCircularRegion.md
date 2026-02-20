**`CLCircularRegion`** — это класс в фреймворке **[[Core Location]]**, который представляет **круговую геозону** (circular geofence) на карте.

Это самая простая и самая часто используемая реализация протокола **[[CLRegion]]**, которая позволяет:
- определять круглую область вокруг точки на карте,
- отслеживать, когда устройство **вошло** или **вышло** из этой зоны,
- запускать локальные уведомления, обновлять данные или выполнять действия при пересечении границы.

### Основные свойства CLCircularRegion

| Свойство        | Тип                        | Обязательно?            | Описание / Пример значения                               |
| --------------- | -------------------------- | ----------------------- | -------------------------------------------------------- |
| `center`        | [[CLLocationCoordinate2D]] | Да                      | Центральная точка круга (широта и долгота)               |
| `radius`        | [[CLLocationDistance]]     | Да                      | Радиус круга в метрах (максимум 100 км в [[iOS]])        |
| `identifier`    | [[String]]                 | Да                      | Уникальный строковый идентификатор зоны (до 36 символов) |
| `notifyOnEntry` | [[Bool]]                   | Нет (по умолчанию true) | Уведомлять при входе в зону                              |
| `notifyOnExit`  | `Bool`                     | Нет (по умолчанию true) | Уведомлять при выходе из зоны                            |

### Ограничения и важные факты (2026 актуально)

- Максимальный радиус — **100 км** (100 000 метров)
- Максимальное количество одновременно мониторимых регионов — **20** на приложение (включая круги и маячки iBeacon)
- Работает в **фоновом режиме** даже при killed-приложении (если включён Background Modes → Location updates)
- Точность срабатывания — обычно 100–500 метров (зависит от [[GPS]], Wi-Fi, сотовой сети)
- Срабатывание — **асинхронное**, может быть задержка 5–60 секунд

### Минимальный рабочий пример (самый частый паттерн 2026)

```swift
import CoreLocation

@MainActor
final class GeofenceManager: NSObject, CLLocationManagerDelegate {
    
    private let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        locationManager.delegate = self
    }
    
    func setupGeofence() {
        // Проверка разрешения
        if locationManager.authorizationStatus == .authorizedAlways ||
           locationManager.authorizationStatus == .authorizedWhenInUse {
            
            let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва, Красная площадь
            let radius: CLLocationDistance = 200 // 200 метров
            
            let region = CLCircularRegion(center: center,
                                         radius: radius,
                                         identifier: "red-square-geofence")
            
            region.notifyOnEntry = true
            region.notifyOnExit = true
            
            locationManager.startMonitoring(for: region)
            
            print("Геозона добавлена: \(region.identifier)")
        } else {
            locationManager.requestAlwaysAuthorization() // или requestWhenInUse
        }
    }
    
    // Делегат: вошли в зону
    func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
        guard let circularRegion = region as? CLCircularRegion else { return }
        print("Вошли в зону:", circularRegion.identifier)
        
        // Показываем уведомление или обновляем UI
        sendLocalNotification(title: "Добро пожаловать!", body: "Вы в зоне Красной площади")
    }
    
    // Делегат: вышли из зоны
    func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
        guard let circularRegion = region as? CLCircularRegion else { return }
        print("Вышли из зоны:", circularRegion.identifier)
    }
    
    // Делегат: ошибка мониторинга
    func locationManager(_ manager: CLLocationManager, monitoringDidFailFor region: CLRegion?, withError error: Error) {
        print("Ошибка мониторинга региона:", error.localizedDescription)
    }
    
    private func sendLocalNotification(title: String, body: String) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        
        let request = UNNotificationRequest(identifier: UUID().uuidString,
                                           content: content,
                                           trigger: nil) // мгновенное
        
        UNUserNotificationCenter.current().add(request)
    }
}
```

### Лучшие практики CLCircularRegion в Swift 2026

- **Всегда** задавайте уникальный `identifier` — нужен для удаления/обновления региона  
- **Включайте** `notifyOnEntry` / `notifyOnExit` в зависимости от задачи  
- **Используйте** радиус **100–500 метров** для большинства сценариев — точность срабатывания выше  
- **Не превышайте 20 регионов** — [[iOS]] просто игнорирует лишние  
- **Для фонового режима** — обязательно включите Background Modes → Location updates и запросите `always` разрешение  
- **Для уведомлений** — комбинируйте с [[UNUserNotificationCenter]] при входе/выходе  
- **В SwiftUI** — создавайте сервис-класс и используйте `.task` для мониторинга  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для [[Core Location]]  
- **Документируйте** — пишите комментарий «CLCircularRegion — геозона радиусом 200 м вокруг Красной площади с уведомлением при входе»

**Короткий итог 2026**:
> `CLCircularRegion` — это **круглая геозона** для мониторинга входа/выхода устройства.  
> В 2026 году:  
> - создаётся через `CLCircularRegion(center:radius:identifier:)`  
> - мониторится через `locationManager.startMonitoring(for:)`  
> - события — `didEnterRegion`, `didExitRegion`  
> - максимум 20 зон, радиус до 100 км  
> Это **единственный** простой и нативный способ реализовать geofencing в iOS-приложении.
