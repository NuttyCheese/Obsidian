**`MKAnnotationView`** — это класс в **[[MapKit]]**, который отвечает за **визуальное представление** аннотации ([[MKAnnotation]]) на карте [[MKMapView]].

Это **не сама аннотация** (координаты + данные), а именно **view**, который рисуется на карте в точке аннотации: стандартный красный пин, кастомная иконка, кластер, callout и т.д.

### Основные свойства MKAnnotationView (самые важные в 2026)

| Свойство                         | Тип                      | Что делает / зачем нужен                             | Самый частый сценарий    |
| -------------------------------- | ------------------------ | ---------------------------------------------------- | ------------------------ |
| `annotation`                     | `MKAnnotation?`          | Связанная аннотация (данные)                         | Обязательно задавать     |
| `image`                          | [[UIImage]]?             | Кастомное изображение вместо стандартного пина       | Замена стандартного пина |
| `glyphImage` / `glyphText`       | `UIImage?` / [[String]]? | Иконка или текст внутри маркера (iOS 11+)            | Современный стиль пинов  |
| `markerTintColor`                | [[UIColor]]?             | Цвет маркера (для [[MKMarkerAnnotationView]])        | Кастомизация цвета       |
| `canShowCallout`                 | [[Bool]]                 | Показывать ли всплывающее окно (callout) при тапе    | Почти всегда `true`      |
| `rightCalloutAccessoryView`      | [[UIView]]?              | Кнопка/иконка справа в callout (обычно [[UIButton]]) | Детали, навигация        |
| `leftCalloutAccessoryView`       | `UIView?`                | Кнопка/иконка слева в callout                        | Аватар, иконка           |
| `detailCalloutAccessoryView`     | `UIView?`                | Полностью кастомный контент вместо title/subtitle    | Сложные callout          |
| `clusterAnnotationView`          | `MKAnnotationView?`      | View для кластера (группы аннотаций)                 | Кластеризация            |
| `centerOffset` / `calloutOffset` | [[CGPoint]]              | Смещение центра пина / callout                       | Точная позиционировка    |

### Самые популярные и рекомендуемые паттерны MKAnnotationView в 2026

#### 1. Стандартный пин с кастомным изображением (самый простой)

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if annotation is MKUserLocation { return nil } // стандартная синяя точка
    
    let identifier = "CustomPin"
    var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
    
    if annotationView == nil {
        annotationView = MKAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        annotationView?.canShowCallout = true
        annotationView?.image = UIImage(systemName: "mappin.circle.fill")?.withTintColor(.systemRed, renderingMode: .alwaysOriginal)
        annotationView?.centerOffset = CGPoint(x: 0, y: -18) // смещение, чтобы пин "стоял" на точке
    } else {
        annotationView?.annotation = annotation
    }
    
    return annotationView
}
```

#### 2. Современный стиль — MKMarkerAnnotationView (iOS 11+)

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if annotation is MKUserLocation { return nil }
    
    let identifier = "MarkerPin"
    var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
    
    if annotationView == nil {
        annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        annotationView?.canShowCallout = true
        annotationView?.markerTintColor = .systemPurple
        annotationView?.glyphText = "Café"           // текст внутри маркера
        annotationView?.glyphImage = UIImage(systemName: "cup.and.saucer.fill")
        annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
    } else {
        annotationView?.annotation = annotation
    }
    
    return annotationView
}
```

#### 3. Полностью кастомный callout (очень популярно для детальных аннотаций)

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    // ... (как выше)
    
    let detailView = UIView(frame: CGRect(x: 0, y: 0, width: 200, height: 80))
    detailView.backgroundColor = .white
    
    let label = UILabel(frame: detailView.bounds.insetBy(dx: 10, dy: 10))
    label.numberOfLines = 0
    label.text = "Ресторан\nАдрес: ул. Ленина, 10\nРейтинг: ★★★★☆"
    label.font = .systemFont(ofSize: 14)
    detailView.addSubview(label)
    
    annotationView?.detailCalloutAccessoryView = detailView
    
    return annotationView
}
```

### Лучшие практики MKAnnotationView в Swift 2026

- **Всегда** используйте **reuse identifier** — `dequeueReusableAnnotationView` экономит память  
- **Для простых пинов** — `MKMarkerAnnotationView` (современный, с glyph и цветом)  
- **Для сложных** — `MKAnnotationView` с `image` или полностью кастомный `detailCalloutAccessoryView`  
- **Не забывайте** `canShowCallout = true` — иначе callout не появится  
- **Для кластеризации** — используйте `MKClusterAnnotation` + `MKAnnotationView` с `clusterAnnotationView`  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` + [[Coordinator]] ([[delegate]])  
- **Документируйте** — пишите комментарий «MKAnnotationView — кастомный пин с изображением и callout для ресторана»

**Короткий итог 2026**:
> `MKAnnotationView` — это **визуальный маркер** на карте, который отображает аннотацию (`MKAnnotation`).  
> В 2026 году:  
> - основной делегат — `mapView(_:viewFor:)`  
> - используйте `MKMarkerAnnotationView` для современного стиля  
> - кастомизируйте `glyphImage`, `markerTintColor`, `calloutAccessoryView`  
> - переиспользуйте через `dequeueReusableAnnotationView`  
> Это **основа** всех маркеров, пинов и точек интереса на картах в iOS-приложениях.
