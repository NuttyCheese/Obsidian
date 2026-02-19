**`MKCircleRenderer`** — это класс в **[[MapKit]]**, который отвечает за **отрисовку** (рендеринг) кругового оверлея ([[MKCircle]]) на карте [[MKMapView]].

Это **стандартный и единственный рекомендуемый** рендерер для всех объектов типа `MKCircle`.  
Без реализации `mapView(_:rendererFor:)` и возврата `MKCircleRenderer` круг просто не отобразится.

### Основные свойства MKCircleRenderer

| Свойство          | Тип            | Значение по умолчанию  | Что контролирует / зачем нужен                |
| ----------------- | -------------- | ---------------------- | --------------------------------------------- |
| `fillColor`       | [[UIColor]]?   | [[nil]] (прозрачно)    | Цвет заливки круга                            |
| `strokeColor`     | `UIColor?`     | `nil` (без обводки)    | Цвет границы круга                            |
| `lineWidth`       | [[CGFloat]]    | 1.0                    | Толщина линии обводки                         |
| `alpha`           | `CGFloat`      | 1.0                    | Общая прозрачность рендерера                  |
| `lineCap`         | [[CGLineCap]]  | `.butt`                | Стиль концов линии                            |
| `lineJoin`        | [[CGLineJoin]] | `.miter`               | Стиль соединения линий (редко влияет на круг) |
| `lineDashPattern` | `[NSNumber]?`  | `nil` (сплошная линия) | Пунктирная/штриховая линия                    |
| `lineDashPhase`   | `CGFloat`      | 0.0                    | Смещение начала пунктира                      |

### Самый популярный и рекомендуемый паттерн (2026 стандарт)

```swift
func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
    if let circle = overlay as? MKCircle {
        let renderer = MKCircleRenderer(circle: circle)
        
        // Основные настройки (самые частые)
        renderer.fillColor   = UIColor.systemBlue.withAlphaComponent(0.25)  // полупрозрачная заливка
        renderer.strokeColor = UIColor.systemBlue                           // яркая граница
        renderer.lineWidth   = 3                                            // толщина обводки
        renderer.alpha       = 0.8                                          // общая прозрачность
        
        // Опционально: пунктирная граница
        // renderer.lineDashPattern = [NSNumber(value: 6), NSNumber(value: 4)]
        
        return renderer
    }
    
    // Для других оверлеев (полилиния, полигон и т.д.)
    return MKOverlayRenderer(overlay: overlay)
}
```

### Полный пример: зона доставки с анимированным кругом

```swift
class DeliveryZoneViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    private var deliveryCircle: MKCircle?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        view = mapView
        
        setupDeliveryZone()
    }
    
    private func setupDeliveryZone() {
        let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва
        let radius: CLLocationDistance = 1500 // 1.5 км
        
        deliveryCircle = MKCircle(center: center, radius: radius)
        deliveryCircle?.title = "Бесплатная доставка"
        deliveryCircle?.subtitle = "В радиусе 1.5 км"
        
        mapView.addOverlay(deliveryCircle!)
        
        // Центрируем карту
        let region = MKCoordinateRegion(center: center,
                                       latitudinalMeters: 4000,
                                       longitudinalMeters: 4000)
        mapView.setRegion(region, animated: true)
    }
    
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        guard let circle = overlay as? MKCircle else {
            return MKOverlayRenderer(overlay: overlay)
        }
        
        let renderer = MKCircleRenderer(circle: circle)
        renderer.fillColor = UIColor.systemGreen.withAlphaComponent(0.2)
        renderer.strokeColor = UIColor.systemGreen
        renderer.lineWidth = 4
        renderer.lineDashPattern = [NSNumber(value: 8), NSNumber(value: 4)] // пунктир
        
        return renderer
    }
}
```

### Лучшие практики MKCircleRenderer в Swift 2026

- **Всегда** возвращайте `MKCircleRenderer` для объектов типа `MKCircle` в методе `rendererFor`  
- **Используйте** полупрозрачную заливку (`withAlphaComponent(0.2–0.3)`) — это стандартный стиль Apple Maps  
- **Добавляйте** `strokeColor` и `lineWidth` — круг без границы выглядит незавершённым  
- **Для пунктира** — `lineDashPattern` — очень удобно для обозначения "виртуальных" зон  
- **Для динамических кругов** — обновляйте радиус/центр через `removeOverlay` + `addOverlay`  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и добавляйте круги в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKCircleRenderer — полупрозрачная зона радиусом 1.5 км с пунктирной границей»

**Короткий итог 2026**:
> `MKCircleRenderer` — это **рендерер** для круговых оверлеев (`MKCircle`) на карте MapKit.  
> В 2026 году:  
> - создаётся в `mapView(_:rendererFor:)`  
> - ключевые свойства — `fillColor`, `strokeColor`, `lineWidth`, `alpha`  
> - идеально для зон доставки, geofencing, радиусов покрытия  
> - поддерживает пунктир (`lineDashPattern`) и полупрозрачность  
> Это **единственный** рекомендуемый способ красиво и правильно отобразить круг на карте.
