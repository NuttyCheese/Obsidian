**`MKPointAnnotation`** — это **самый простой и часто используемый** класс в **[[MapKit]]**, который реализует протокол **[[MKAnnotation]]**.

Он представляет **точечную аннотацию** (маркер, pin) на карте [[MKMapView]] с минимальным набором свойств: координаты + заголовок + подзаголовок.

Это **базовый класс** для большинства случаев, когда нужна простая метка без сложной логики или кастомных данных.

### Основные свойства MKPointAnnotation

| Свойство     | Тип                        | Обязательно? | Описание / Пример использования |
| ------------ | -------------------------- | ------------ | ------------------------------- |
| `coordinate` | [[CLLocationCoordinate2D]] | Да           | Географическая точка на карте   |
| `title`      | String?                    | Нет          | Основной заголовок (в callout)  |
| `subtitle`   | `String?`                  | Нет          | Подзаголовок (в callout)        |

### Самые популярные способы создания и использования MKPointAnnotation (2026 стандарт)

#### 1. Простейшая аннотация (самый частый случай)

```swift
let annotation = MKPointAnnotation()
annotation.coordinate = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва, Красная площадь
annotation.title = "Красная площадь"
annotation.subtitle = "Сердце Москвы"

mapView.addAnnotation(annotation)
```

#### 2. Удобный инициализатор с параметрами (рекомендуемый)

```swift
let annotation = MKPointAnnotation(
    coordinate: CLLocationCoordinate2D(latitude: 40.7128, longitude: -74.0060),
    title: "Статуя Свободы",
    subtitle: "Нью-Йорк, США"
)

mapView.addAnnotation(annotation)
```

#### 3. Полный пример с добавлением нескольких аннотаций и центрированием

```swift
class MapViewController: UIViewController, MKMapViewDelegate {
    
    private let mapView = MKMapView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        mapView.delegate = self
        view = mapView
        
        addAnnotations()
    }
    
    private func addAnnotations() {
        let places = [
            (name: "Эйфелева башня", coord: CLLocationCoordinate2D(latitude: 48.8584, longitude: 2.2945)),
            (name: "Колизей", coord: CLLocationCoordinate2D(latitude: 41.8902, longitude: 12.4922)),
            (name: "Большой театр", coord: CLLocationCoordinate2D(latitude: 55.7602, longitude: 37.6184))
        ]
        
        var annotations: [MKPointAnnotation] = []
        
        for place in places {
            let annotation = MKPointAnnotation()
            annotation.coordinate = place.coord
            annotation.title = place.name
            annotations.append(annotation)
        }
        
        mapView.addAnnotations(annotations)
        
        // Центрируем карту на первую точку
        if let first = annotations.first {
            let region = MKCoordinateRegion(center: first.coordinate,
                                           latitudinalMeters: 1000000,
                                           longitudinalMeters: 1000000)
            mapView.setRegion(region, animated: true)
        }
    }
    
    // Кастомизация вида (опционально)
    func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
        if annotation is MKUserLocation { return nil }
        
        let identifier = "SimplePin"
        var view = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
        
        if view == nil {
            view = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
            view?.canShowCallout = true
        } else {
            view?.annotation = annotation
        }
        
        return view
    }
}
```

### Лучшие практики MKPointAnnotation в Swift 2026

- **Используйте** `MKPointAnnotation` для **простых случаев** (координаты + название)  
- **Для сложных данных** — создавайте собственный класс, реализующий `MKAnnotation`  
- **Всегда** задавайте `title` — это основной текст в callout  
- **Не забывайте** `canShowCallout = true` в `viewFor annotation:` — иначе callout не появится  
- **Для кастомного вида** — используйте `MKMarkerAnnotationView` (glyph, цвет) или `MKAnnotationView` с `image`  
- **Для кластеризации** — реализуйте `MKClusterAnnotation` и кастомный кластер-view  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и добавляйте аннотации в `updateUIView`  
- **Документируйте** — пишите комментарий «MKPointAnnotation — простая метка для достопримечательности»

**Короткий итог 2026**:
> `MKPointAnnotation` — это **самый простой и самый популярный** класс для создания маркеров на карте.  
> В 2026 году:  
> - минимум — `coordinate` (обязательно) + `title` / `subtitle` (очень желательно)  
> - используйте как базовый класс или напрямую  
> - кастомизируйте вид через `MKMarkerAnnotationView` или `MKAnnotationView` в делегате  
> - это **основа** всех простых точек интереса (POI) на картах в iOS-приложениях  

Удачи с точными и информативными маркерами на картах в твоём проекте! 📍