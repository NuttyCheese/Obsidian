**`MKCoordinateSpan`** — это простая структура в **[[MapKit]]**, которая описывает **размер видимой области** карты в терминах угловых градусов.

Она используется исключительно внутри **[[MKCoordinateRegion]]** и определяет, насколько широко/высоко растянута карта по широте и долготе.

### Основные свойства структуры MKCoordinateSpan

```swift
public struct MKCoordinateSpan {
    public var latitudeDelta:  CLLocationDegrees   // разница широты (север-юг) в градусах
    public var longitudeDelta: CLLocationDegrees   // разница долготы (запад-восток) в градусах
}
```

- `latitudeDelta` — сколько градусов широты занимает видимая область (от центра вверх и вниз)
- `longitudeDelta` — сколько градусов долготы занимает видимая область (от центра влево и вправо)

### Типичные значения latitudeDelta / longitudeDelta (2026 ориентиры)

| Уровень зума              | latitudeDelta / longitudeDelta | Примерный масштаб на экране          | Когда использовать |
|---------------------------|--------------------------------|---------------------------------------|---------------------|
| Очень близко (улица)      | 0.001–0.005                    | ~100–500 метров                       | Детальный вид, адрес |
| Городской масштаб         | 0.01–0.05                      | ~1–5 км                               | Центр города, район |
| Область/пригород          | 0.1–0.5                        | ~10–50 км                             | Область доставки, пригород |
| Регион/страна             | 1–10                           | ~100–1000 км                          | Страна, крупный регион |
| Весь мир                  | 90–180                         | Показывает почти всю планету          | Глобус, обзорный вид |

**Важно**:  
`longitudeDelta` на разных широтах означает разное реальное расстояние!  
На экваторе 1° долготы ≈ 111 км, а на широте 60° (Санкт-Петербург) — уже ≈ 55 км.

### Самые популярные способы создания MKCoordinateSpan

#### 1. Ручное задание (классика)

```swift
let span = MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)  // ≈ 5–6 км в диаметре

let region = MKCoordinateRegion(
    center: CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173),
    span: span
)

mapView.setRegion(region, animated: true)
```

#### 2. Через реальные метры (самый рекомендуемый и удобный способ в 2026)

```swift
let region = MKCoordinateRegion(
    center: CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194),  // Сан-Франциско
    latitudinalMeters: 3000,     // ±1.5 км по широте (север-юг)
    longitudinalMeters: 3000     // ±1.5 км по долготе (запад-восток)
)

mapView.setRegion(region, animated: true)
```

**Почему метры лучше градусов**:  
- Интуитивно понятны (метры, а не градусы)  
- Автоматически учитывают кривизну Земли и широту  
- Результат всегда примерно одинаковый независимо от текущего положения

#### 3. Динамический зум под контент (маршрут, аннотации)

```swift
// После добавления всех аннотаций или оверлеев
let rect = mapView.visibleMapRect  // или route.polyline.boundingMapRect
mapView.setVisibleMapRect(rect, edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50), animated: true)
```

### Лучшие практики MKCoordinateSpan в Swift 2026

- **Предпочитайте** конструктор с `latitudinalMeters` / `longitudinalMeters` — это самый понятный и надёжный способ  
- **Не используйте** слишком маленькие значения (`< 0.0001`) — карта может "зависнуть" или показывать артефакты  
- **Для маршрутов** — всегда берите `route.boundingMapRect` + `edgePadding`  
- **Для динамического зума** — используйте `MKMapViewCameraZoomRange` (iOS 13+) в связке с `MKMapCamera`  
- **В SwiftUI** — `Map` автоматически управляет регионом, но можно задать `MapCameraPosition.region`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ при использовании MapKit  
- **Документируйте** — пишите комментарий «MKCoordinateSpan — видимая область ±2 км по широте и долготе»

**Короткий итог 2026**:
> `MKCoordinateSpan` — это **размер видимой области карты** в градусах (широта и долгота).  
> В 2026 году:  
> - задаётся внутри `MKCoordinateRegion`  
> - самый удобный способ — `latitudinalMeters` / `longitudinalMeters`  
> - используется в `setRegion`, `setVisibleMapRect`, [[CLCircularRegion]] и т.д.  
> Это **самый простой** и **самый часто встречающийся** способ управлять масштабом карты.
