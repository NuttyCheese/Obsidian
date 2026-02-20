**`CLLocationDegrees`** — это **тип-алиас** (псевдоним типа) в фреймворке **Core Location**, который используется для представления **угловых значений** широты и долготы в градусах.

```swift
typealias CLLocationDegrees = Double
```

По сути — это просто `Double`, но с **явной семантикой** географических координат.

### Основные характеристики и диапазоны (актуально на 2026 год)

| Значение              | Диапазон допустимых значений          | Что означает                              | Примечание |
|-----------------------|----------------------------------------|-------------------------------------------|------------|
| `latitude`            | –90.0 … +90.0 градусов                 | Широта: юг (−) → север (+)                | –90° = Южный полюс, +90° = Северный полюс |
| `longitude`           | –180.0 … +180.0 градусов               | Долгота: запад (−) → восток (+)           | –180° и +180° — одна и та же линия (антимеридиан) |
| Максимальная точность | ~15 знаков после запятой (Double)      | ~1 см на поверхности Земли                | Реальная точность GPS обычно 3–10 м |

### Самые частые способы использования CLLocationDegrees

#### 1. Создание координат (самый популярный паттерн)

```swift
let moscow = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
let tokyo  = CLLocationCoordinate2D(latitude: 35.6762, longitude: 139.6503)
let sydney = CLLocationCoordinate2D(latitude: -33.8688, longitude: 151.2093)
```

#### 2. Получение текущих координат пользователя

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.last else { return }
    
    let lat: CLLocationDegrees = location.coordinate.latitude
    let lon: CLLocationDegrees = location.coordinate.longitude
    
    print("Широта: \(lat), Долгота: \(lon)")
}
```

#### 3. Форматированный вывод (очень полезно для UI и логов)

```swift
extension CLLocationCoordinate2D {
    var formatted: String {
        let formatter = NumberFormatter()
        formatter.maximumFractionDigits = 6
        formatter.minimumFractionDigits = 6
        
        let latStr = formatter.string(from: NSNumber(value: latitude)) ?? "—"
        let lonStr = formatter.string(from: NSNumber(value: longitude)) ?? "—"
        
        let ns = latitude >= 0 ? "N" : "S"
        let ew = longitude >= 0 ? "E" : "W"
        
        return String(format: "%.6f°%@ %.6f°%@", abs(latitude), ns, abs(longitude), ew)
    }
}

let coord = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)
print(coord.formatted)  // → "55.755800°N 37.617300°E"
```

#### 4. Проверка валидности координат (рекомендуется всегда)

```swift
func isValidCoordinate(_ coord: CLLocationCoordinate2D) -> Bool {
    return (-90...90).contains(coord.latitude) &&
           (-180...180).contains(coord.longitude)
}
```

### Лучшие практики CLLocationDegrees в Swift 2026

- **Всегда** проверяйте диапазон перед использованием — недопустимые значения могут привести к краху карты или неверным расчётам  
- **Предпочитайте** `CLLocationCoordinate2D` как единый тип вместо отдельных `Double` — это типобезопаснее и читаемее  
- **Для отображения** — используйте форматирование с 6 знаками после запятой (стандарт точности GPS)  
- **Для расчёта расстояний** — всегда преобразуйте в `CLLocation` → `distance(from:)`  
- **В SwiftUI** — передавайте `CLLocationCoordinate2D` напрямую в `Map` / `MapMarker`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании Core Location  
- **Документируйте** — пишите комментарий «CLLocationDegrees — широта и долгота центра Москвы (Красная площадь)»

**Короткий итог 2026**:
> `CLLocationDegrees` — это просто `Double`, но с **явной географической семантикой** (широта/долгота в градусах).  
> В 2026 году:  
> - диапазон широты: –90…+90  
> - диапазон долготы: –180…+180  
> - используется **везде** в Core Location и MapKit: координаты, регионы, аннотации, маршруты  
> - всегда проверяйте валидность перед использованием  
> Это **самый базовый** и **самый часто встречающийся** тип для любой работы с геолокацией.

Удачи с точными и безопасными координатами в твоём проекте! 🌍📍