**`MKPolygon`** — это класс в фреймворке **[[MapKit]]**, который реализует протокол **[[MKOverlay]]** и представляет **замкнутый полигон** (многоугольник) на карте [[MKMapView]].

Это самый удобный способ отобразить на карте:
- административные границы (страны, регионы, города),
- зоны покрытия (доставка, сигнал),
- полигоны зданий,
- территории парков, озёр, заповедников,
- выделенные области поиска или геозоны,
- любые кастомные замкнутые фигуры (включая полигоны с дырками).

### Основные свойства MKPolygon

| Свойство                | Тип                                            | Обязательно?       | Описание / Пример использования                 |
| ----------------------- | ---------------------------------------------- | ------------------ | ----------------------------------------------- |
| `coordinate`            | [[CLLocationCoordinate2D]]                     | Да                 | Центр полигона (вычисляется автоматически)      |
| `boundingMapRect`       | [[MKMapRect]]                                  | Да (автоматически) | Прямоугольник, охватывающий весь полигон        |
| `points` / `pointCount` | UnsafeMutablePointer<[[MKMapPoint]]> / [[Int]] | Да                 | Массив координат вершин полигона                |
| `interiorPolygons`      | [ [[MKPolygon]] ]?                             | Нет                | Внутренние полигоны (дырки в основном полигоне) |
| `title` / `subtitle`    | [[String]]?                                    | Нет                | Заголовок и подзаголовок (для callout)          |

### Способы создания MKPolygon (самые популярные в 2026)

#### 1. Простой полигон (4–6 точек)

```swift
let coordinates = [
    CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173),  // Красная площадь
    CLLocationCoordinate2D(latitude: 55.7600, longitude: 37.6200),
    CLLocationCoordinate2D(latitude: 55.7580, longitude: 37.6250),
    CLLocationCoordinate2D(latitude: 55.7530, longitude: 37.6220),
    CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173)   // замыкаем
]

let polygon = MKPolygon(coordinates: coordinates, count: coordinates.count)
polygon.title = "Центр Москвы"
polygon.subtitle = "Примерная зона"

mapView.addOverlay(polygon)
```

#### 2. Полный пример с дыркой (полигон с внутренним полигоном)

```swift
// Внешний полигон (большой прямоугольник)
let outerCoords = [
    CLLocationCoordinate2D(latitude: 55.75, longitude: 37.60),
    CLLocationCoordinate2D(latitude: 55.76, longitude: 37.60),
    CLLocationCoordinate2D(latitude: 55.76, longitude: 37.63),
    CLLocationCoordinate2D(latitude: 55.75, longitude: 37.63),
    CLLocationCoordinate2D(latitude: 55.75, longitude: 37.60)
]

let outer = MKPolygon(coordinates: outerCoords, count: outerCoords.count)

// Внутренний полигон (дырка)
let innerCoords = [
    CLLocationCoordinate2D(latitude: 55.753, longitude: 37.610),
    CLLocationCoordinate2D(latitude: 55.755, longitude: 37.610),
    CLLocationCoordinate2D(latitude: 55.755, longitude: 37.615),
    CLLocationCoordinate2D(latitude: 55.753, longitude: 37.615),
    CLLocationCoordinate2D(latitude: 55.753, longitude: 37.610)
]

let inner = MKPolygon(coordinates: innerCoords, count: innerCoords.count)

// Создаём полигон с дыркой
let polygonWithHole = MKPolygon(coordinates: outerCoords, count: outerCoords.count, interiorPolygons: [inner])

polygonWithHole.title = "Зона с вырезом"

mapView.addOverlay(polygonWithHole)
```

#### 3. Отрисовка полигона (обязательный делегат)

```swift
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let polygon = overlay as? MKPolygon {
        let renderer = MKPolygonRenderer(polygon: polygon)
        
        renderer.fillColor = UIColor.systemBlue.withAlphaComponent(0.25)
        renderer.strokeColor = UIColor.systemBlue
        renderer.lineWidth = 3
        renderer.alpha = 0.8
        
        // Для полигона с дыркой fillRule автоматически .evenOdd
        // но можно явно задать
        renderer.fillRule = .evenOdd
        
        return renderer
    }
    
    return MKOverlayRenderer(overlay: overlay)
}
```

### Лучшие практики MKPolygon в Swift 2026

- **Всегда** реализуйте `mapView(_:rendererFor:)` — без него полигон не отобразится  
- **Используйте** `MKPolygonRenderer` — это стандартный и самый удобный рендерер  
- **Для зон с дырками** — передавайте внутренние полигоны в `interiorPolygons`  
- **Для производительности** — не создавайте полигоны с тысячами точек (MapKit сам оптимизирует)  
- **Для динамических полигонов** — обновляйте через `removeOverlay` + `addOverlay`  
- **В SwiftUI** — оборачивайте `MKMapView` в [[UIViewRepresentable]] и добавляйте полигоны в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKPolygon — зона доставки радиусом 2 км с полупрозрачной заливкой»

**Короткий итог 2026**:
> `MKPolygon` — это **замкнутый полигон** на карте MapKit для отображения зон, границ, территорий и геозон.  
> В 2026 году:  
> - создаётся через `MKPolygon(coordinates:count:)` или с `interiorPolygons` для дырок  
> - добавляется через `mapView.addOverlay(polygon)`  
> - рендерится в `mapView(_:rendererFor:)` с помощью `MKPolygonRenderer`  
> - поддерживает title/subtitle и callout  
> Это **самый гибкий** способ показать произвольные замкнутые области на карте.
