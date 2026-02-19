**`MKDirections`** — это класс в фреймворке **[[MapKit]]**, который отвечает за **построение маршрутов** (directions) между двумя точками на карте Apple Maps.

Он позволяет:
- рассчитывать автомобильные, пешие, велосипедные и общественным транспортом маршруты,
- получать альтернативные варианты,
- учитывать пробки (traffic),
- получать расстояние, время в пути, инструкции поворотов,
- отображать маршрут на карте как полилинию ([[MKPolyline]]).

### Основные свойства и методы MKDirections (актуально на 2026 год)

| Свойство / Метод                              | Тип / Возвращает                            | Описание / Зачем нужен                                    | Самый частый сценарий  |
| --------------------------------------------- | ------------------------------------------- | --------------------------------------------------------- | ---------------------- |
| `MKDirections.Request`                        | —                                           | Конфигурация запроса (обязательно)                        | Начало любого маршрута |
| `source` / `destination`                      | [[MKMapItem]]                               | Точки старта и финиша                                     | Обязательные поля      |
| `transportType`                               | [[MKDirectionsTransportType]]               | `.automobile`, `.walking`, `.transit`, `.any`             | Выбор типа маршрута    |
| `requestsAlternateRoutes`                     | [[Bool]]                                    | Запрашивать ли альтернативные маршруты (до 3–5 вариантов) | Показ нескольких путей |
| `calculate(completionHandler:)`               | [[MKDirections]] → `MKDirections.Response?` | Асинхронный расчёт маршрута                               | Основной метод         |
| `Response.routes`                             | [ [[MKRoute]] ]                             | Массив найденных маршрутов                                | Получение результатов  |
| `MKRoute` → `polyline`                        | [[MKPolyline]]                              | Полилиния маршрута для добавления на карту                | Отрисовка на MKMapView |
| `MKRoute` → `distance` / `expectedTravelTime` | [[CLLocationDistance]] / [[TimeInterval]]   | Длина пути (метры) и время в пути (секунды)               | Отображение информации |

### Минимальный рабочий пример построения маршрута (современный стиль 2026)

```swift
import MapKit

@MainActor
class RouteViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        view = mapView
        
        calculateRoute()
    }
    
    private func calculateRoute() {
        let startCoordinate = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва, Красная площадь
        let endCoordinate   = CLLocationCoordinate2D(latitude: 59.9343, longitude: 30.3351) // Санкт-Петербург
        
        let request = MKDirections.Request()
        request.source = MKMapItem(placemark: MKPlacemark(coordinate: startCoordinate))
        request.destination = MKMapItem(placemark: MKPlacemark(coordinate: endCoordinate))
        request.transportType = .automobile
        request.requestsAlternateRoutes = true  // альтернативные маршруты
        
        let directions = MKDirections(request: request)
        directions.calculate { [weak self] response, error in
            guard let self else { return }
            
            guard let route = response?.routes.first else {
                print("Ошибка построения маршрута:", error?.localizedDescription ?? "Неизвестно")
                return
            }
            
            // Очищаем старые оверлеи
            self.mapView.removeOverlays(self.mapView.overlays)
            
            // Добавляем маршрут как полилинию
            self.mapView.addOverlay(route.polyline)
            
            // Центрируем карту на маршруте
            self.mapView.setVisibleMapRect(route.polyline.boundingMapRect,
                                          edgePadding: UIEdgeInsets(top: 50, left: 50, bottom: 50, right: 50),
                                          animated: true)
            
            // Показываем информацию
            print("Расстояние: \(route.distance / 1000) км")
            print("Время в пути: \(Int(route.expectedTravelTime / 60)) мин")
        }
    }
    
    // Обязательный делегат для отрисовки маршрута
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        if let polyline = overlay as? MKPolyline {
            let renderer = MKPolylineRenderer(polyline: polyline)
            renderer.strokeColor = .systemBlue
            renderer.lineWidth = 6
            renderer.alpha = 0.9
            return renderer
        }
        return MKOverlayRenderer(overlay: overlay)
    }
}
```

### Полезные кастомизации MKPolylineRenderer

```swift
let renderer = MKPolylineRenderer(polyline: route.polyline)
renderer.strokeColor = UIColor.systemGreen
renderer.lineWidth = 8
renderer.lineCap = .round
renderer.lineJoin = .round
renderer.alpha = 0.85
renderer.lineDashPattern = [NSNumber(value: 6), NSNumber(value: 4)] // пунктир
```

### Лучшие практики MKDirections в Swift 2026

- **Всегда** проверяйте `response?.routes.first` — может быть `nil` при ошибке или отсутствии маршрута  
- **Используйте** `requestsAlternateRoutes = true` — даёт до 3–5 вариантов пути  
- **Обрабатывайте** ошибки в `completionHandler` — показывайте пользователю алерт  
- **Для отображения** — добавляйте `polyline` через `addOverlay` и рендерьте в `rendererFor overlay`  
- **Для нескольких маршрутов** — показывайте все `routes` и позволяйте выбрать  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и запускайте `MKDirections` в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKDirections — построение автомобильного маршрута с отображением полилинии»

**Короткий итог 2026**:
> `MKDirections` — это **API для построения маршрутов** в Apple Maps.  
> В 2026 году:  
> - создаётся через `MKDirections.Request` + `calculate`  
> - результат — массив `MKRoute` с `polyline`  
> - отображается как `MKPolyline` через `addOverlay` и `rendererFor`  
> - поддерживает автомобиль, пеший, велосипед, транспорт  
> Это **единственный** нативный способ построить маршрут внутри iOS-приложения.

Удачи с точными и красивыми маршрутами в твоём приложении! 🛣️