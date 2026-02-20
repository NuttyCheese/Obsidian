**`CLLocationDirection`** — это **тип-алиас** (псевдоним типа) в фреймворке **[[Core Location]]**, который используется для представления **азимута** (направления, курса, heading) в градусах.

```swift
typealias CLLocationDirection = Double
```

По сути — это просто `Double`, но с явной семантикой **угла направления** относительно севера.

### Основные характеристики CLLocationDirection (2026 актуально)

| Характеристика              | Диапазон / Значение                  | Что означает                                                                 | Примечание |
|-----------------------------|--------------------------------------|------------------------------------------------------------------------------|------------|
| Диапазон                    | 0.0 … 360.0 градусов                 | 0° = север, 90° = восток, 180° = юг, 270° = запад                           | 0° и 360° — одно и то же |
| Точность                    | ~0.1–1° (зависит от компаса устройства) | Реальная точность магнитного компаса обычно ±5–15°                           | Лучше в магнитном режиме |
| Магнитное vs Истинное       | Магнитное (по умолчанию) vs Истинное | `CLLocationManager.headingFilter`, `trueHeading` vs `magneticHeading`        | Истинное требует калибровки |
| Специальное значение        | `-1`                                 | Направление неизвестно (до первого обновления или при ошибке)                | Проверяйте перед использованием |

### Самые популярные способы получения и использования CLLocationDirection

#### 1. Получение текущего направления (heading) через [[CLLocationManager]]

```swift
let locationManager = CLLocationManager()

// Включаем обновление heading
locationManager.startUpdatingHeading()

func locationManager(_ manager: CLLocationManager, didUpdateHeading newHeading: CLHeading) {
    let magneticDirection: CLLocationDirection = newHeading.magneticHeading   // магнитное
    let trueDirection: CLLocationDirection = newHeading.trueHeading          // истинное (если доступно)
    
    if trueDirection >= 0 {
        print("Истинное направление: \(trueDirection)°")
    } else {
        print("Магнитное направление: \(magneticDirection)° (истинное недоступно)")
    }
    
    // Пример: поворот иконки компаса
    compassImageView.transform = CGAffineTransform(rotationAngle: -CGFloat(magneticDirection * .pi / 180))
}
```

#### 2. Использование в [[MKMapView]] (поворот карты по направлению движения)

```swift
mapView.userTrackingMode = .followWithHeading  // автоматически поворачивает карту по heading

// Или вручную через MKMapCamera
let camera = MKMapCamera()
camera.centerCoordinate = userLocation.coordinate
camera.heading = userHeading.magneticHeading   // поворот карты
camera.pitch = 60
camera.altitude = 500

mapView.setCamera(camera, animated: true)
```

#### 3. Расчёт направления между двумя точками (bearing)

```swift
extension CLLocationCoordinate2D {
    func bearing(to point: CLLocationCoordinate2D) -> CLLocationDirection {
        let lat1 = latitude * .pi / 180
        let lon1 = longitude * .pi / 180
        let lat2 = point.latitude * .pi / 180
        let lon2 = point.longitude * .pi / 180
        
        let dLon = lon2 - lon1
        
        let y = sin(dLon) * cos(lat2)
        let x = cos(lat1) * sin(lat2) - sin(lat1) * cos(lat2) * cos(dLon)
        
        var bearing = atan2(y, x) * 180 / .pi
        bearing = (bearing + 360).truncatingRemainder(dividingBy: 360)
        
        return bearing
    }
}

let moscow = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
let spb    = CLLocationCoordinate2D(latitude: 59.9343, longitude: 30.3351)

let direction = moscow.bearing(to: spb)
print("Направление Москва → Санкт-Петербург: \(direction)°")  // ≈ 329° (северо-северо-запад)
```

### Лучшие практики CLLocationDirection в Swift 2026

- **Всегда** проверяйте `newHeading.trueHeading >= 0` — если < 0, значит истинное направление недоступно (нет калибровки или [[GPS]])  
- **Используйте** `.round` для отображения (округляйте до 0–1 знака после запятой)  
- **Для UI** — преобразуйте в читаемый формат (N, NE, E, SE и т.д.)

```swift
extension CLLocationDirection {
    var cardinalDirection: String {
        let directions = ["С", "СВ", "В", "ЮВ", "Ю", "ЮЗ", "З", "СЗ"]
        let index = Int((self + 22.5).truncatingRemainder(dividingBy: 360) / 45)
        return directions[index]
    }
}

print(45.0.cardinalDirection)   // "СВ"
print(0.0.cardinalDirection)    // "С"
```

- **В SwiftUI** — используйте `CLLocationDirection` напрямую в `Map` или `CompassView`  
- **Для AR** — синхронизируйте с `ARSession` и `CLHeading` для точного направления  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании Core Location  
- **Документируйте** — пишите комментарий «CLLocationDirection — текущее магнитное направление устройства (0° = север)»

**Короткий итог 2026**:
> `CLLocationDirection` — это просто `Double`, но с **явной семантикой** направления в градусах (0° = север, по часовой стрелке).  
> В 2026 году:  
> - диапазон 0…360°  
> - получайте из `CLHeading.magneticHeading` / `trueHeading`  
> - используйте для поворота карты, компаса, навигации, AR  
> - всегда проверяйте `trueHeading >= 0` перед использованием истинного направления  
> Это **самый базовый** тип для работы с направлением и ориентацией устройства.
