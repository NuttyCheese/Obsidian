**`MKClusterAnnotation`** — это класс в фреймворке **[[MapKit]]** (доступен с iOS 11+), который автоматически создаётся системой при **кластеризации** (группировке) аннотаций на карте `MKMapView`.

Когда на карте слишком много аннотаций в одной области (при большом зуме), MapKit может **сгруппировать** их в один кластерный маркер — именно это и есть экземпляр `MKClusterAnnotation`.

### Когда и как работает кластеризация

Кластеризация включается автоматически, если:

- Вы явно разрешили её у аннотаций через свойство `MKAnnotationView.clusteringIdentifier`
- На экране много аннотаций (обычно >10–20 в видимой области)
- Зум карты недостаточно большой, чтобы показать все пины отдельно

### Основные свойства MKClusterAnnotation

| Свойство            | Тип                        | Описание                                             | Самый частый сценарий     |
| ------------------- | -------------------------- | ---------------------------------------------------- | ------------------------- |
| `memberAnnotations` | [ [[MKAnnotation]] ]       | Массив всех аннотаций, которые входят в этот кластер | Отображение количества    |
| `title`             | [[String]]?                | Автоматически генерируется (например, "5")           | Количество аннотаций      |
| `subtitle`          | `String?`                  | Обычно [[nil]] или кастомное значение                | Дополнительная информация |
| `coordinate`        | [[CLLocationCoordinate2D]] | Средняя координата всех членов кластера              | Центр кластера            |

### Как включить кластеризацию (рекомендуемый паттерн 2026)

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if annotation is MKUserLocation { return nil }
    
    // Это обычная аннотация
    let identifier = "CafePin"
    var annotationView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
    
    if annotationView == nil {
        annotationView = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
        annotationView?.canShowCallout = true
        annotationView?.markerTintColor = .systemOrange
        annotationView?.glyphImage = UIImage(systemName: "cup.and.saucer.fill")
        
        // Самое важное — включаем кластеризацию
        annotationView?.clusteringIdentifier = "CafeCluster"  // ← любое уникальное имя
    } else {
        annotationView?.annotation = annotation
    }
    
    return annotationView
}
```

Теперь MapKit автоматически:
- сгруппирует аннотации с одинаковым `clusteringIdentifier`
- создаст `MKClusterAnnotation` вместо отдельных пинов
- вызовет `viewFor annotation:` с `annotation` типа `MKClusterAnnotation`

### Как кастомизировать вид кластера

```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if let cluster = annotation as? MKClusterAnnotation {
        let identifier = "ClusterView"
        var clusterView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
        
        if clusterView == nil {
            clusterView = MKMarkerAnnotationView(annotation: cluster, reuseIdentifier: identifier)
            clusterView?.canShowCallout = true
            
            // Кастомизация кластера
            clusterView?.markerTintColor = .systemPurple
            clusterView?.glyphText = "\(cluster.memberAnnotations.count)"  // показываем количество
            // или
            // clusterView?.glyphImage = UIImage(systemName: "building.2.fill")
            
            // Кастомный callout
            let label = UILabel()
            label.text = "Здесь \(cluster.memberAnnotations.count) кафе"
            label.font = .systemFont(ofSize: 14)
            clusterView?.detailCalloutAccessoryView = label
        } else {
            clusterView?.annotation = cluster
        }
        
        return clusterView
    }
    
    // ... обычная обработка одиночных аннотаций (как выше)
}
```

### Лучшие практики MKClusterAnnotation в Swift 2026

- **Всегда** задавайте одинаковый `clusteringIdentifier` у всех похожих аннотаций  
- **Используйте** `MKMarkerAnnotationView` для кластеров — поддерживает `glyphText` и `glyphImage`  
- **Показывайте количество** через `cluster.memberAnnotations.count` в `glyphText` или `title`  
- **Для кастомного вида** — проверяйте `annotation is MKClusterAnnotation` и возвращайте специальный view  
- **Не забывайте** `canShowCallout = true` — иначе пользователь не увидит количество/детали  
- **Для большого количества точек** — кластеризация сильно улучшает производительность и UX  
- **В SwiftUI** — оборачивайте `MKMapView` в `UIViewRepresentable` и обрабатывайте кластеры в `viewFor`  
- **Документируйте** — пишите комментарий «MKClusterAnnotation — группировка кафе в кластер с отображением количества»

**Короткий итог 2026**:
> `MKClusterAnnotation` — это **автоматически созданный кластер** из нескольких аннотаций при сильном зуме.  
> В 2026 году:  
> - включается через `clusteringIdentifier` в `MKAnnotationView`  
> - распознаётся в `viewFor annotation:` как `MKClusterAnnotation`  
> - содержит `memberAnnotations` — все аннотации внутри кластера  
> - идеально кастомизируется через `glyphText` / `glyphImage` / `detailCalloutAccessoryView`  
> Это **ключевой** инструмент для чистой и производительной карты при большом количестве маркеров.
