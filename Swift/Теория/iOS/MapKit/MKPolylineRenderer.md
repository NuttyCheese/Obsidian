**`MKPolylineRenderer`** — это класс в **[[MapKit]]**, который отвечает за **отрисовку** (рендеринг) ломаных линий ([[MKPolyline]]) на карте [[MKMapView]].

Это **единственный рекомендуемый** рендерер для всех объектов типа `MKPolyline` (маршруты, треки, границы, дороги и любые линейные оверлеи).  
Без возврата `MKPolylineRenderer` из метода делегата `mapView(_:rendererFor:)` линия просто не появится на карте.

### Основные свойства MKPolylineRenderer

| Свойство          | Тип            | Значение по умолчанию  | Что контролирует / зачем нужен              |
| ----------------- | -------------- | ---------------------- | ------------------------------------------- |
| `strokeColor`     | [[UIColor]]?   | [[nil]] (без обводки)  | Цвет линии (обязательно задавать)           |
| `lineWidth`       | [[CGFloat]]    | 1.0                    | Толщина линии (в пунктах)                   |
| `alpha`           | `CGFloat`      | 1.0                    | Общая прозрачность линии                    |
| `lineCap`         | [[CGLineCap]]  | `.butt`                | Стиль концов линии                          |
| `lineJoin`        | [[CGLineJoin]] | `.miter`               | Стиль соединения углов                      |
| `lineDashPattern` | `[NSNumber]?`  | `nil` (сплошная линия) | Пунктир/штриховка (массив длин сегментов)   |
| `lineDashPhase`   | `CGFloat`      | 0.0                    | Смещение начала пунктира                    |
| `miterLimit`      | `CGFloat`      | 10.0                   | Ограничение длины острых углов при `.miter` |

### Самый популярный и рекомендуемый паттерн (2026 стандарт)

```swift
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    guard let polyline = overlay as? MKPolyline else {
        return MKOverlayRenderer(overlay: overlay)
    }
    
    let renderer = MKPolylineRenderer(polyline: polyline)
    
    // Самые частые и красивые настройки
    renderer.strokeColor = UIColor.systemBlue
    renderer.lineWidth   = 6
    renderer.lineCap     = .round
    renderer.lineJoin    = .round
    renderer.alpha       = 0.9
    
    // Опционально: пунктирная линия
    // renderer.lineDashPattern = [NSNumber(value: 8), NSNumber(value: 4)]
    
    return renderer
}
```

### Полный пример: отображение маршрута из MKDirections

```swift
func showRoute(from start: CLLocationCoordinate2D, to end: CLLocationCoordinate2D) {
    let request = MKDirections.Request()
    request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
    request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
    request.transportType = .automobile
    
    let directions = MKDirections(request: request)
    directions.calculate { response, error in
        guard let route = response?.routes.first else { return }
        
        // Добавляем маршрут как оверлей
        self.mapView.addOverlay(route.polyline)
        
        // Центрируем карту на весь маршрут с отступами
        self.mapView.setVisibleMapRect(route.polyline.boundingMapRect,
                                      edgePadding: UIEdgeInsets(top: 80, left: 80, bottom: 80, right: 80),
                                      animated: true)
    }
}

// Рендерер (в делегате)
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let polyline = overlay as? MKPolyline {
        let renderer = MKPolylineRenderer(polyline: polyline)
        renderer.strokeColor = .systemBlue
        renderer.lineWidth = 8
        renderer.lineCap = .round
        renderer.lineJoin = .round
        renderer.alpha = 0.85
        
        return renderer
    }
    return MKOverlayRenderer(overlay: overlay)
}
```

### Лучшие практики MKPolylineRenderer в Swift 2026

- **Всегда** возвращайте `MKPolylineRenderer` для объектов `MKPolyline` в методе `rendererFor`  
- **Используйте** `lineCap = .round` и `lineJoin = .round` — это даёт самый современный и плавный вид  
- **Задавайте** `strokeColor` и `lineWidth` (обычно 4–8 pt) — без них линия будет невидимой или слишком тонкой  
- **Для выделения активного маршрута** — анимируйте `strokeEnd` или меняйте `strokeColor` / `lineWidth`  
- **Для пунктира** — `lineDashPattern` — отлично подходит для "альтернативных" или "рекомендованных" маршрутов  
- **Для производительности** — не создавайте слишком много полилиний одновременно (MapKit оптимизирует)  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и задавайте рендереры в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKPolylineRenderer — отображение маршрута с толщиной 6 pt, закруглёнными концами и соединениями»

**Короткий итог 2026**:
> `MKPolylineRenderer` — это **рендерер** для ломаных линий (`MKPolyline`) на карте MapKit.  
> В 2026 году:  
> - создаётся в методе `mapView(_:rendererFor:)`  
> - ключевые свойства — `strokeColor`, `lineWidth`, `lineCap`, `lineJoin`, `alpha`  
> - идеально для маршрутов, треков, границ, дорог  
> - это **единственный** рекомендуемый способ красиво и правильно отобразить линию на карте  
