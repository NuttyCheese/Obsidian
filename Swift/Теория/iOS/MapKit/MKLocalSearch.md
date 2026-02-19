**`MKLocalSearch`** — это класс в фреймворке **MapKit**, который позволяет выполнять **поиск мест** (points of interest, POI) на карте Apple Maps.

С его помощью можно искать:
- кафе, рестораны, магазины,
- адреса, улицы, города,
- достопримечательности,
- организации по названию или категории,
- результаты в заданном регионе карты.

Это **самый мощный и рекомендуемый** способ локального поиска внутри iOS-приложений (замена устаревшему `MKLocalSearchRequest` + `MKLocalSearch` в старых версиях).

### Основные свойства и методы MKLocalSearch (актуально на 2026 год)

| Свойство / Метод                             | Тип / Возвращает                              | Описание / Зачем нужен                                      | Самый частый сценарий |
|----------------------------------------------|-----------------------------------------------|-------------------------------------------------------------|------------------------|
| `MKLocalSearch.Request`                      | —                                             | Конфигурация запроса (обязательно)                          | Начало любого поиска   |
| `naturalLanguageQuery`                       | `String?`                                     | Поисковый запрос на естественном языке ("кофе рядом", "пиццерия Москва") | Основной ввод пользователя |
| `pointOfInterestFilter`                      | `MKPointOfInterestFilter?`                    | Фильтр по категориям (кафе, отели, парки и т.д.)            | Ограничение результатов |
| `region`                                     | `CLCircularRegion?` / `MKCoordinateRegion`    | Ограничение области поиска                                  | Поиск в видимой области карты |
| `init(request:)`                             | —                                             | Создание объекта поиска                                     | — |
| `start(completionHandler:)`                  | `MKLocalSearch` → `MKLocalSearch.Response?`   | Асинхронный запуск поиска                                   | Основной метод         |
| `Response.mapItems`                          | `[MKMapItem]`                                 | Массив найденных мест (адрес, название, координаты и т.д.)  | Получение результатов  |
| `Response.boundingRegion`                    | `MKCoordinateRegion?`                         | Рекомендуемая область для центрирования карты               | Автоматический зум     |

### Минимальный рабочий пример поиска (современный стиль 2026)

```swift
import MapKit

@MainActor
class SearchViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    private let searchBar = UISearchBar()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        mapView.delegate = self
        view.addSubview(mapView)
        // ... constraints
        
        searchBar.delegate = self
        navigationItem.titleView = searchBar
    }
    
    private func performSearch(query: String) {
        let request = MKLocalSearch.Request()
        request.naturalLanguageQuery = query
        request.region = mapView.region  // поиск в видимой области
        
        // Опционально: фильтр по категориям
        request.pointOfInterestFilter = MKPointOfInterestFilter(including: [.cafe, .restaurant])
        
        let search = MKLocalSearch(request: request)
        search.start { [weak self] response, error in
            guard let self else { return }
            
            guard let response else {
                print("Ошибка поиска:", error?.localizedDescription ?? "Неизвестно")
                return
            }
            
            // Очищаем старые аннотации
            self.mapView.removeAnnotations(self.mapView.annotations)
            
            // Добавляем новые
            let annotations = response.mapItems.map { item -> MKPointAnnotation in
                let annotation = MKPointAnnotation()
                annotation.coordinate = item.placemark.coordinate
                annotation.title = item.name
                annotation.subtitle = item.phoneNumber ?? item.placemark.title
                return annotation
            }
            
            self.mapView.addAnnotations(annotations)
            
            // Центрируем карту на результатах
            if let boundingRegion = response.boundingRegion {
                self.mapView.setRegion(boundingRegion, animated: true)
            }
        }
    }
}

extension SearchViewController: UISearchBarDelegate {
    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        guard let query = searchBar.text, !query.isEmpty else { return }
        performSearch(query: query)
        searchBar.resignFirstResponder()
    }
}
```

### Полезные кастомизации MKLocalSearch.Request

```swift
let request = MKLocalSearch.Request()
request.naturalLanguageQuery = "кофейня"
request.resultTypes = [.pointOfInterest, .address]  // только POI и адреса
request.pointOfInterestFilter = MKPointOfInterestFilter(excluding: [.store])  // исключить магазины
request.region = MKCoordinateRegion(center: userLocation.coordinate,
                                    latitudinalMeters: 5000,
                                    longitudinalMeters: 5000)
```

### Лучшие практики MKLocalSearch в Swift 2026

- **Всегда** ограничивайте поиск регионом (`request.region = mapView.region`) — результаты будут релевантнее  
- **Используйте** `pointOfInterestFilter` — сильно уменьшает количество мусора в результатах  
- **Обрабатывайте** ошибки и пустой `response` — показывайте пользователю сообщение  
- **Для отображения** — создавайте `MKPointAnnotation` из `MKMapItem` и добавляйте через `addAnnotations`  
- **Для автодополнения** — используйте `MKLocalSearchCompleter` (отдельный класс)  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и запускайте поиск в `updateUIView`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для MapKit  
- **Документируйте** — пишите комментарий «MKLocalSearch — поиск кофеен в видимой области карты»

**Короткий итог 2026**:
> `MKLocalSearch` — это **API для поиска мест** (POI, адресов) в Apple Maps.  
> В 2026 году:  
> - создаётся через `MKLocalSearch.Request` + `naturalLanguageQuery`  
> - запускается асинхронно через `start`  
> - результат — массив `MKMapItem` с названием, адресом, координатами  
> - отображается как аннотации (`MKPointAnnotation`) на `MKMapView`  
> Это **единственный** нативный способ реализовать поиск по карте в iOS-приложении.

Удачи с точным и удобным поиском мест в твоём приложении! 🔍🗺️