**`MKMarkerAnnotationView`** — это современный класс в **MapKit** (доступен с iOS 11+), который представляет **стилизованный маркер** (annotation view) на карте `MKMapView`.

Это **рекомендуемая замена** старому `MKPinAnnotationView` — более гибкий, красивый и кастомизируемый вариант пина с поддержкой цвета, глифа (иконки/текста внутри), анимаций и кластеризации.

### Основные преимущества MKMarkerAnnotationView

- Современный плоский дизайн (как в Apple Maps)
- Автоматическая поддержка **тёмной темы**
- Встроенный глиф (текст или иконка внутри маркера)
- Легко менять цвет маркера (`markerTintColor`)
- Поддержка **кластеризации** (группировка маркеров)
- Анимированное появление/исчезновение
- Callout с кнопками и кастомным контентом

### Основные свойства MKMarkerAnnotationView

| Свойство                     | Тип                  | Значение по умолчанию                  | Что контролирует / зачем нужен |
|------------------------------|----------------------|----------------------------------------|--------------------------------|
| `markerTintColor`            | `UIColor?`           | `nil` (системный синий)                | Цвет маркера                   |
| `glyphImage`                 | `UIImage?`           | `nil`                                  | Кастомная иконка внутри маркера |
| `glyphText`                  | `String?`            | `nil`                                  | Текст внутри маркера (1–2 символа) |
| `selectedGlyphImage`         | `UIImage?`           | `nil`                                  | Иконка при выбранном состоянии |
| `animatesWhenAdded`          | `Bool`               | `true`                                 | Анимированное появление маркера |
| `canShowCallout`             | `Bool`               | `true`                                 | Показывать ли всплывающее окно |
| `rightCalloutAccessoryView`  | `UIView?`            | `nil`                                  | Кнопка справа в callout        |
| `detailCalloutAccessoryView` | `UIView?`            | `nil`                                  | Полностью кастомный контент callout |

### Рекомендуемый паттерн использования в 2026 году

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if annotation is MKUserLocation { return nil } // стандартная синяя точка
    
    let identifier = "MarkerPin"
    var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
    
    if annotationView == nil {
        annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        
        // Основные настройки
        annotationView?.canShowCallout = true
        annotationView?.animatesWhenAdded = true
        
        // Кастомизация цвета
        annotationView?.markerTintColor = .systemPurple
        
        // Глиф — текст или иконка
        annotationView?.glyphText = "Café"                    // текст внутри маркера
        // или
        annotationView?.glyphImage = UIImage(systemName: "cup.and.saucer.fill")?.withTintColor(.white, renderingMode: .alwaysOriginal)
        
        // Callout
        annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
        // или полностью кастомный
        // annotationView?.detailCalloutAccessoryView = customDetailView
    } else {
        annotationView?.annotation = annotation
    }
    
    return annotationView
}
```

### Полный пример с анимированным маркером и callout

```swift
class CafeAnnotation: NSObject, MKAnnotation {
    let coordinate: CLLocationCoordinate2D
    let name: String
    let address: String
    
    var title: String? { name }
    var subtitle: String? { address }
    
    init(coordinate: CLLocationCoordinate2D, name: String, address: String) {
        self.coordinate = coordinate
        self.name = name
        self.address = address
        super.init()
    }
}

// В делегате
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    guard let cafe = annotation as? CafeAnnotation else { return nil }
    
    let identifier = "CafePin"
    var view = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
    
    if view == nil {
        view = MKMarkerAnnotationView(annotation: cafe, reuseIdentifier: identifier)
        view?.canShowCallout = true
        view?.markerTintColor = .systemOrange
        view?.glyphImage = UIImage(systemName: "cup.and.saucer")?.withTintColor(.white)
        
        // Кастомный callout
        let detailView = UIView(frame: CGRect(x: 0, y: 0, width: 180, height: 60))
        let label = UILabel(frame: detailView.bounds.insetBy(dx: 8, dy: 8))
        label.numberOfLines = 0
        label.text = "\(cafe.name)\n\(cafe.address)"
        label.font = .systemFont(ofSize: 14)
        detailView.addSubview(label)
        
        view?.detailCalloutAccessoryView = detailView
        
        // Кнопка справа
        let button = UIButton(type: .detailDisclosure)
        view?.rightCalloutAccessoryView = button
    } else {
        view?.annotation = cafe
    }
    
    return view
}
```

### Лучшие практики MKMarkerAnnotationView в Swift 2026

- **Всегда** используйте `dequeueReusableAnnotationView` — экономит память  
- **Предпочитайте** `MKMarkerAnnotationView` над старым `MKAnnotationView` — современный стиль, лучше поддержка тёмной темы  
- **Для текста внутри** — используйте `glyphText` (1–2 символа) или `glyphImage`  
- **Для цвета** — `markerTintColor` автоматически адаптируется к тёмной теме  
- **Для callout** — добавляйте `rightCalloutAccessoryView` (кнопка) или `detailCalloutAccessoryView` (кастомный контент)  
- **Для кластеризации** — реализуйте `MKClusterAnnotation` и кастомный кластер-view  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` + `Coordinator`  
- **Документируйте** — пишите комментарий «MKMarkerAnnotationView — современный пин с глифом и кастомным callout для кафе»

**Короткий итог 2026**:
> `MKMarkerAnnotationView` — это **современный стиль маркера** на карте MapKit (с 2017 года — основной выбор).  
> В 2026 году:  
> - используйте вместо старого `MKAnnotationView`  
> - ключевые свойства — `markerTintColor`, `glyphImage`/`glyphText`, `canShowCallout`  
> - кастомизируйте callout через `rightCalloutAccessoryView` / `detailCalloutAccessoryView`  
> - переиспользуйте через `dequeueReusableAnnotationView`  
> Это **самый красивый** и **самый рекомендуемый** способ отображения точек на карте в iOS-приложениях.

Удачи с стильными, информативными и современными маркерами на картах в твоём проекте! 📍✨