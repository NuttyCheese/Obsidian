**`MKOverlayRenderer`** — это **базовый класс** в **[[MapKit]]**, который отвечает за **отрисовку** (рендеринг) любых объектов, соответствующих протоколу **[[MKOverlay]]** (линии, полигоны, круги и кастомные оверлеи) на карте [[MKMapView]].

Это **единственный** способ задать, как именно будет выглядеть оверлей на экране: цвет заливки, толщина линии, прозрачность, пунктир, градиенты и т.д.

### Ключевые факты о MKOverlayRenderer (2026 актуально)

| Характеристика       | Описание                                                                                                               | Важные замечания                        |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Наследование         | `MKOverlayRenderer` — базовый класс<br>Конкретные: [[MKPolylineRenderer]], [[MKPolygonRenderer]], [[MKCircleRenderer]] | Всегда используйте конкретные рендереры |
| Создаётся в делегате | `mapView(_:rendererFor:)` — обязательный метод делегата                                                                | Без него оверлей не отобразится         |
| Переиспользование    | Не поддерживается (в отличие от аннотаций)                                                                             | Каждый раз создаётся новый экземпляр    |
| Производительность   | Рендерится на [[GPU]] → очень быстро, даже при большом количестве оверлеев                                             | Оптимизируйте сложные пути              |
| Поддержка анимации   | Можно анимировать свойства рендерера (fillColor, strokeColor, lineWidth)                                               | Через [[CABasicAnimation]]              |

### Основные свойства базового MKOverlayRenderer

| Свойство                     | Тип                  | Значение по умолчанию                  | Что контролирует |
|------------------------------|----------------------|----------------------------------------|------------------|
| `alpha`                      | `CGFloat`            | 1.0                                    | Общая прозрачность |
| `blendMode`                  | `CGBlendMode`        | `.normal`                              | Режим смешивания (редко меняют) |
| `overlay`                    | `MKOverlay`          | —                                      | Связанный оверлей (чтение) |
| `contentScaleFactor`         | `CGFloat`            | 1.0                                    | Масштаб для Retina (обычно не трогают) |

### Конкретные рендереры (самые используемые)

| Класс рендерера        | Для какого оверлея | Основные кастомные свойства                                          | Пример использования              |
| ---------------------- | ------------------ | -------------------------------------------------------------------- | --------------------------------- |
| [[MKPolylineRenderer]] | `MKPolyline`       | `strokeColor`, `lineWidth`, `lineCap`, `lineJoin`, `lineDashPattern` | Маршруты, треки                   |
| [[MKPolygonRenderer]]  | `MKPolygon`        | `fillColor`, `strokeColor`, `lineWidth`, `fillRule`                  | Зоны, границы, полигоны с дырками |
| [[MKCircleRenderer]]   | `MKCircle`         | `fillColor`, `strokeColor`, `lineWidth`, `lineDashPattern`           | Радиус доставки, geofencing       |

### Рекомендуемый паттерн (самый частый и современный)

```swift
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let polyline = overlay as? MKPolyline {
        let renderer = MKPolylineRenderer(polyline: polyline)
        
        renderer.strokeColor = UIColor.systemBlue
        renderer.lineWidth = 6
        renderer.lineCap = .round
        renderer.lineJoin = .round
        renderer.alpha = 0.9
        
        // Пунктир (опционально)
        // renderer.lineDashPattern = [NSNumber(value: 8), NSNumber(value: 4)]
        
        return renderer
    }
    
    if let circle = overlay as? MKCircle {
        let renderer = MKCircleRenderer(circle: circle)
        
        renderer.fillColor   = UIColor.systemGreen.withAlphaComponent(0.25)
        renderer.strokeColor = UIColor.systemGreen
        renderer.lineWidth   = 4
        renderer.alpha       = 0.8
        
        return renderer
    }
    
    if let polygon = overlay as? MKPolygon {
        let renderer = MKPolygonRenderer(polygon: polygon)
        
        renderer.fillColor   = UIColor.systemPurple.withAlphaComponent(0.3)
        renderer.strokeColor = UIColor.systemPurple
        renderer.lineWidth   = 3
        renderer.fillRule    = .evenOdd  // для дырок
        
        return renderer
    }
    
    // Для кастомных оверлеев
    return MKOverlayRenderer(overlay: overlay)
}
```

### Лучшие практики MKOverlayRenderer в Swift 2026

- **Всегда** проверяйте тип оверлея через `as?` и возвращайте конкретный рендерер  
- **Используйте** полупрозрачную заливку (`withAlphaComponent(0.2–0.3)`) — это стандартный стиль Apple Maps  
- **Добавляйте** `strokeColor` и `lineWidth` — без обводки оверлей часто выглядит незавершённым  
- **Для пунктира** — `lineDashPattern` — отлично подходит для "виртуальных" зон или границ  
- **Для анимации** — анимируйте свойства рендерера (`strokeColor`, `lineWidth`, `alpha`) через `CABasicAnimation`  
- **Для производительности** — не создавайте слишком сложные пути — MapKit сам оптимизирует рендеринг  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и задавайте рендереры в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKPolylineRenderer — отображение маршрута с толщиной 6 pt и закруглёнными концами»

**Короткий итог 2026**:
> `MKOverlayRenderer` — это **класс для отрисовки** оверлеев на карте MapKit.  
> В 2026 году:  
> - создаётся в методе `mapView(_:rendererFor:)`  
> - конкретные типы: `MKPolylineRenderer`, `MKPolygonRenderer`, `MKCircleRenderer`  
> - ключевые свойства: `strokeColor`, `fillColor`, `lineWidth`, `alpha`, `lineDashPattern`  
> - это **единственный** способ задать внешний вид линий, кругов, полигонов и других оверлеев  
