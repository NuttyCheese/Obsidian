**`MKPolyline`** — это класс в фреймворке **[[MapKit]]**, который реализует протокол **`MKOverlay`** и представляет **ломаную линию** (polyline) на карте [[MKMapView]].

Это **самый популярный** способ отобразить на карте:
- автомобильные/пешие маршруты (из [[MKDirections]]),
- треки (GPS-трекинг, бег, велосипед),
- границы административных единиц,
- линии электропередач, трубопроводы,
- любые линейные объекты (дороги, реки, границы зон).

### Основные свойства MKPolyline

| Свойство             | Тип                                | Обязательно?       | Описание / Пример использования                    |
| -------------------- | ---------------------------------- | ------------------ | -------------------------------------------------- |
| `points`             | `UnsafeMutablePointer<MKMapPoint>` | Да                 | Массив координат в проекции карты ([[MKMapPoint]]) |
| `pointCount`         | [[Int]]                            | Да                 | Количество точек в массиве `points`                |
| `coordinate`         | [[CLLocationCoordinate2D]]         | Да (автоматически) | Центр полигона (вычисляется)                       |
| `boundingMapRect`    | [[MKMapRect]]                      | Да (автоматически) | Прямоугольник, охватывающий весь путь              |
| `title` / `subtitle` | [[String]]?                        | Нет                | Заголовок и подзаголовок (для callout)             |

### Способы создания MKPolyline (самые популярные в 2026)

#### 1. Из массива координат (самый простой)

```swift
let coordinates = [
    CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173),  // Москва
    CLLocationCoordinate2D(latitude: 55.7600, longitude: 37.6200),
    CLLocationCoordinate2D(latitude: 55.7650, longitude: 37.6250),
    CLLocationCoordinate2D(latitude: 55.7700, longitude: 37.6300)
]

let polyline = MKPolyline(coordinates: coordinates, count: coordinates.count)
polyline.title = "Прогулочный маршрут"
polyline.subtitle = "4 км"

mapView.addOverlay(polyline)
```

#### 2. Самый частый случай — из маршрута MKDirections

```swift
func showRoute(from start: CLLocationCoordinate2D, to end: CLLocationCoordinate2D) {
    let request = MKDirections.Request()
    request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
    request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
    request.transportType = .automobile
    
    let directions = MKDirections(request: request)
    directions.calculate { response, error in
        guard let route = response?.routes.first else { return }
        
        // Добавляем маршрут как полилинию
        self.mapView.addOverlay(route.polyline)
        
        // Центрируем карту на весь маршрут
        self.mapView.setVisibleMapRect(route.polyline.boundingMapRect,
                                      edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                                      animated: true)
        
        print("Длина маршрута: \(route.distance / 1000) км")
        print("Время в пути: \(Int(route.expectedTravelTime / 60)) мин")
    }
}
```

#### 3. Отрисовка полилинии (обязательный делегат)

```swift
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let polyline = overlay as? MKPolyline {
        let renderer = MKPolylineRenderer(polyline: polyline)
        
        renderer.strokeColor = UIColor.systemBlue
        renderer.lineWidth = 6
        renderer.lineCap = .round
        renderer.lineJoin = .round
        renderer.alpha = 0.9
        
        // Пунктирная линия (опционально)
        // renderer.lineDashPattern = [NSNumber(value: 6), NSNumber(value: 4)]
        
        return renderer
    }
    
    return MKOverlayRenderer(overlay: overlay)
}
```

### Полезные кастомизации [[MKPolylineRenderer]]

```swift
let renderer = MKPolylineRenderer(polyline: polyline)
renderer.strokeColor = UIColor.systemGreen
renderer.lineWidth = 8
renderer.lineCap = .round
renderer.lineJoin = .round
renderer.alpha = 0.85
renderer.lineDashPattern = [NSNumber(value: 8), NSNumber(value: 4)] // пунктир
```

### Лучшие практики MKPolyline в Swift 2026

- **Всегда** реализуйте `mapView(_:rendererFor:)` — без него полилиния не отобразится  
- **Используйте** `MKPolylineRenderer` — это стандартный и самый удобный рендерер  
- **Для маршрутов** — берите готовую `MKRoute.polyline` из `MKDirections`  
- **Для динамических линий** — обновляйте через `removeOverlay` + `addOverlay`  
- **Для производительности** — не добавляйте слишком много точек (MapKit сам оптимизирует)  
- **В [[SwiftUI]]** — оборачивайте `MKMapView` в [[UIViewRepresentable]] и добавляйте полилинии в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKPolyline — отображение автомобильного маршрута с толщиной 6 pt и закруглёнными концами»

**Короткий итог 2026**:
> `MKPolyline` — это **ломаная линия** на карте MapKit для отображения маршрутов, треков, границ и любых линейных объектов.  
> В 2026 году:  
> - создаётся через `MKPolyline(coordinates:count:)` или берётся из `MKRoute.polyline`  
> - добавляется через `mapView.addOverlay(polyline)`  
> - рендерится в `mapView(_:rendererFor:)` с помощью `MKPolylineRenderer`  
> - поддерживает title/subtitle и callout  
> Это **самый популярный** способ показать путь, маршрут или трек на карте.
