**`CLLocationDistance`** — это **тип-алиас** (псевдоним типа) в фреймворке **[[Core Location]]**, который используется для представления **расстояния** в метрах.

```swift
typealias CLLocationDistance = Double
```

По сути — это просто `Double`, но с явной семантикой **линейного расстояния** на поверхности Земли.

### Основные характеристики CLLocationDistance (актуально на 2026 год)

| Характеристика              | Диапазон / Значение                  | Что означает                                                                 | Примечание |
|-----------------------------|--------------------------------------|------------------------------------------------------------------------------|------------|
| Единица измерения           | Метры (m)                            | Все расстояния в Core Location — в метрах                                   | Никогда не путайте с километрами |
| Точность                    | ~0.01–1 м (зависит от источника)     | GPS обычно даёт точность 3–10 м, Wi-Fi/сотовые — хуже                      | Реальная точность редко лучше 1 м |
| Специальное значение        | `-1` или `Double.nan`                | Расстояние неизвестно / не удалось вычислить                                | Проверяйте перед использованием |
| Максимальное значение       | Теоретически до ~20 000 000 м (половина окружности Земли) | На практике редко превышает 10 000 км (международные перелёты)             | — |

### Самые популярные способы получения и использования CLLocationDistance

#### 1. Расстояние между двумя точками (самый частый сценарий)

```swift
let coord1 = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва
let coord2 = CLLocationCoordinate2D(latitude: 59.9343, longitude: 30.3351) // Санкт-Петербург

let location1 = CLLocation(latitude: coord1.latitude, longitude: coord1.longitude)
let location2 = CLLocation(latitude: coord2.latitude, longitude: coord2.longitude)

let distance: CLLocationDistance = location1.distance(from: location2)

print("Расстояние: \(distance) метров")
print("Расстояние: \(distance / 1000) км")  // ≈ 634 км
```

#### 2. Точность текущего местоположения

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.last else { return }
    
    let accuracy: CLLocationDistance = location.horizontalAccuracy   // точность по горизонтали
    
    if accuracy < 0 {
        print("Точность неизвестна")
    } else if accuracy < 10 {
        print("Высокая точность (±\(accuracy) м)")
    } else {
        print("Точность: ±\(accuracy) м")
    }
}
```

#### 3. Расстояние до ближайшей аннотации / POI

```swift
func findNearestCafe(userLocation: CLLocation, cafes: [CafeAnnotation]) -> (CafeAnnotation, CLLocationDistance)? {
    var nearest: (CafeAnnotation, CLLocationDistance)?
    
    for cafe in cafes {
        let cafeLocation = CLLocation(latitude: cafe.coordinate.latitude,
                                     longitude: cafe.coordinate.longitude)
        
        let dist = userLocation.distance(from: cafeLocation)
        
        if nearest == nil || dist < nearest!.1 {
            nearest = (cafe, dist)
        }
    }
    
    return nearest
}
```

#### 4. Форматированный вывод расстояния (очень полезно для UI)

```swift
extension CLLocationDistance {
    var formatted: String {
        if self < 1000 {
            return String(format: "%.0f м", self)
        } else if self < 100_000 {
            return String(format: "%.1f км", self / 1000)
        } else {
            return String(format: "%.0f км", self / 1000)
        }
    }
}

let distance: CLLocationDistance = 2345.7
print(distance.formatted)  // → "2.3 км"
```

### Лучшие практики CLLocationDistance в Swift 2026

- **Всегда** проверяйте `horizontalAccuracy >= 0` — если < 0, расстояние ненадёжно  
- **Для отображения** — используйте человекочитаемый формат (м → км, округление)  
- **Для сравнения** — используйте `distance(from:)` — это учитывает кривизну Земли  
- **Не путайте** с `verticalAccuracy` — это высота над уровнем моря  
- **В SwiftUI** — передавайте `CLLocationDistance` в `Text` с форматированием  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании Core Location  
- **Документируйте** — пишите комментарий «CLLocationDistance — расстояние до ближайшего кафе (в метрах)»

**Короткий итог 2026**:
> `CLLocationDistance` — это просто `Double`, но с **явной семантикой** расстояния в метрах.  
> В 2026 году:  
> - единица измерения — **метры** (всегда)  
> - получайте через `location.distance(from:)` или `horizontalAccuracy`  
> - используйте для: расстояний между точками, точности локации, зон покрытия  
> - всегда форматируйте перед показом пользователю (м → км)  
> Это **самый базовый** и **самый часто встречающийся** тип для любых измерений расстояния в геолокационных приложениях.
