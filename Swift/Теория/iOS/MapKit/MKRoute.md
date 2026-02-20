**`MKRoute`** — это класс в фреймворке **[[MapKit]]**, который представляет **один рассчитанный маршрут** (route) между двумя точками, полученный из **`MKDirections`**.

Это **самый важный** результат работы `MKDirections.calculate`, содержащий всю необходимую информацию о пути:
- геометрию (полилинию),
- длину,
- время в пути,
- пошаговые инструкции,
- альтернативные варианты (если запрошены),
- тип транспорта и т.д.

### Основные свойства MKRoute (самые используемые в 2026)

| Свойство             | Тип                           | Описание / Пример значения                             | Самый частый сценарий           |
| -------------------- | ----------------------------- | ------------------------------------------------------ | ------------------------------- |
| `polyline`           | [[MKPolyline]]                | Полная геометрия маршрута (ломаная линия)              | Добавление на карту как оверлей |
| `distance`           | [[CLLocationDistance]]        | Длина маршрута в метрах                                | Отображение "X км"              |
| `expectedTravelTime` | [[TimeInterval]]              | Ожидаемое время в пути в секундах                      | Отображение "Y мин"             |
| `transportType`      | [[MKDirectionsTransportType]] | Тип транспорта (`.automobile`, `.walking`, `.transit`) | Проверка и отображение          |
| `steps`              | `[MKRoute.Step]`              | Массив пошаговых инструкций (повороты, названия дорог) | Пошаговая навигация             |
| `name`               | [[String]]                    | Название маршрута (например, "M-10")                   | Отображение в UI                |
| `advisoryNotices`    | `[String]`                    | Предупреждения (платные дороги, паромы и т.д.)         | Показывать пользователю         |
| `hasTrafficIncident` | [[Bool]]                      | Есть ли пробки/инциденты на маршруте                   | Визуальное выделение            |

### Самый популярный и рекомендуемый паттерн (2026 стандарт)

```swift
import MapKit

@MainActor
class RouteViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        view = mapView
        
        calculateAndShowRoute()
    }
    
    private func calculateAndShowRoute() {
        let start = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва
        let end   = CLLocationCoordinate2D(latitude: 59.9343, longitude: 30.3351) // СПб
        
        let request = MKDirections.Request()
        request.source = MKMapItem(placemark: MKPlacemark(coordinate: start))
        request.destination = MKMapItem(placemark: MKPlacemark(coordinate: end))
        request.transportType = .automobile
        request.requestsAlternateRoutes = true
        
        let directions = MKDirections(request: request)
        directions.calculate { [weak self] response, error in
            guard let self else { return }
            
            guard let route = response?.routes.first else {
                print("Ошибка маршрута:", error?.localizedDescription ?? "Нет маршрута")
                return
            }
            
            // Очищаем старые оверлеи
            self.mapView.removeOverlays(self.mapView.overlays)
            
            // Добавляем маршрут
            self.mapView.addOverlay(route.polyline)
            
            // Центрируем карту на весь маршрут с отступами
            self.mapView.setVisibleMapRect(route.polyline.boundingMapRect,
                                          edgePadding: UIEdgeInsets(top: 80, left: 80, bottom: 80, right: 80),
                                          animated: true)
            
            // Показываем информацию
            let distanceKm = route.distance / 1000
            let timeMin = Int(route.expectedTravelTime / 60)
            
            print("Маршрут: \(route.name ?? "Без названия")")
            print("Длина: \(String(format: "%.1f", distanceKm)) км")
            print("Время: \(timeMin) мин")
            print("Пробки/инциденты: \(route.hasTrafficIncident ? "Да" : "Нет")")
            
            // Если есть альтернативы — можно показать их тоже
            for altRoute in response?.routes.dropFirst() ?? [] {
                self.mapView.addOverlay(altRoute.polyline)
                print("Альтернатива: \(altRoute.distance / 1000) км")
            }
        }
    }
    
    // Обязательный рендерер для отображения линии
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        guard let polyline = overlay as? MKPolyline else {
            return MKOverlayRenderer(overlay: overlay)
        }
        
        let renderer = MKPolylineRenderer(polyline: polyline)
        renderer.strokeColor = .systemBlue
        renderer.lineWidth = 8
        renderer.lineCap = .round
        renderer.lineJoin = .round
        renderer.alpha = 0.9
        
        return renderer
    }
}
```

### Лучшие практики MKRoute в Swift 2026

- **Всегда** берите `routes.first` — это основной (самый быстрый/короткий) маршрут  
- **Проверяйте** `response?.routes` — может быть пустым при ошибке или недоступности  
- **Добавляйте** `polyline` как оверлей и **центрируйте** карту через `boundingMapRect` с `edgePadding`  
- **Отображайте** `distance` и `expectedTravelTime` — это ключевые метрики для пользователя  
- **Для альтернатив** — используйте `requestsAlternateRoutes = true` и показывайте все `routes`  
- **Для пошаговой навигации** — используйте `route.steps` (каждый шаг — `MKRoute.Step` с инструкцией, расстоянием и временем)  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и добавляйте `polyline` в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с [[iOS]] 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKRoute — рассчитанный автомобильный маршрут с полилинией и временем в пути»

**Короткий итог 2026**:
> `MKRoute` — это **один рассчитанный маршрут** от `MKDirections`, содержащий полилинию, расстояние, время и инструкции.  
> В 2026 году:  
> - главный результат `MKDirections.calculate`  
> - ключевые свойства — `polyline`, `distance`, `expectedTravelTime`, `steps`  
> - добавляется на карту как `MKPolyline` через `addOverlay`  
> - рендерится через `MKPolylineRenderer` в делегате  
> Это **основа** всей навигации и отображения путей в iOS-приложениях.
