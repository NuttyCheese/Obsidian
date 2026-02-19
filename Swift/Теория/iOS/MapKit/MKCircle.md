**`MKCircle`** — это класс в фреймворке **[[MapKit]]**, который представляет **круговую область** (circular overlay) на карте [[MKMapView]].

Это самый простой и часто используемый способ отобразить на карте:
- зону покрытия (например, радиус доставки),
- геозону (geofencing),
- область поиска,
- индикатор расстояния вокруг точки,
- визуализацию сигнала Wi-Fi/Bluetooth и т.д.

### Основные свойства MKCircle

| Свойство             | Тип                        | Описание                               | Самый частый сценарий |
| -------------------- | -------------------------- | -------------------------------------- | --------------------- |
| `coordinate`         | [[CLLocationCoordinate2D]] | Центр круга (обязательно)              | Точка интереса        |
| `radius`             | [[CLLocationDistance]]     | Радиус круга в метрах (обязательно)    | Размер зоны           |
| `title` / `subtitle` | [[String]]?                | Заголовок и подзаголовок (для callout) | Информация о зоне     |
| `overlayRenderer`    | [[MKOverlayRenderer]]?     | Рендерер (обычно `MKCircleRenderer`)   | Кастомизация вида     |

### Самые популярные способы создания MKCircle (2026 стандарт)

#### 1. Простейший круг вокруг точки

```swift
let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва, Красная площадь
let circle = MKCircle(center: center, radius: 500) // 500 метров

mapView.addOverlay(circle)
```

#### 2. Полный пример с кастомным рендерером (самый частый паттерн)

```swift
class MapViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        
        // Добавляем круг
        let center = CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194) // Сан-Франциско
        let circle = MKCircle(center: center, radius: 1000) // 1 км
        circle.title = "Зона доставки"
        circle.subtitle = "Бесплатно в радиусе 1 км"
        
        mapView.addOverlay(circle)
        
        // Центрируем карту на круге
        let region = MKCoordinateRegion(center: center,
                                       latitudinalMeters: 3000,
                                       longitudinalMeters: 3000)
        mapView.setRegion(region, animated: true)
    }
    
    // Обязательный делегат для рендеринга оверлея
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        if let circle = overlay as? MKCircle {
            let renderer = MKCircleRenderer(circle: circle)
            renderer.fillColor = UIColor.systemBlue.withAlphaComponent(0.2)
            renderer.strokeColor = UIColor.systemBlue
            renderer.lineWidth = 3
            renderer.alpha = 0.7
            
            return renderer
        }
        
        return MKOverlayRenderer(overlay: overlay)
    }
}
```

### Полезные кастомизации [[MKCircleRenderer]]

```swift
let renderer = MKCircleRenderer(circle: circle)
renderer.fillColor   = UIColor.systemGreen.withAlphaComponent(0.25)  // полупрозрачная заливка
renderer.strokeColor = UIColor.systemGreen                                // цвет обводки
renderer.lineWidth   = 4                                                  // толщина границы
renderer.lineDashPattern = [NSNumber(value: 5), NSNumber(value: 5)]       // пунктирная линия
renderer.alpha       = 0.8                                                // общая прозрачность
```

### Лучшие практики MKCircle в Swift 2026

- **Всегда** реализуйте `mapView(_:rendererFor:)` — без него круг не отобразится  
- **Используйте** `MKCircleRenderer` — это стандартный и самый удобный рендерер для кругов  
- **Добавляйте** `title` / `subtitle` — они отображаются в callout при тапе на круг  
- **Для динамических кругов** — обновляйте `radius` и вызывайте `mapView.removeOverlay(circle)` + `addOverlay(circle)`  
- **Для нескольких кругов** — храните их в массиве и добавляйте/удаляйте группами  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и добавляйте оверлеи в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKCircle — зона покрытия радиусом 1 км с полупрозрачной заливкой»

**Короткий итог 2026**:
> `MKCircle` — это **круглый оверлей** на карте MapKit для отображения радиусов, зон, покрытия и т.д.  
> В 2026 году:  
> - создаётся через `MKCircle(center:radius:)`  
> - отображается через `mapView.addOverlay(circle)`  
> - кастомизируется в `mapView(_:rendererFor:)` с помощью `MKCircleRenderer`  
> - поддерживает `title`/`subtitle` и callout  
> Это **самый простой** и **самый часто используемый** способ показать круговую область на карте.
