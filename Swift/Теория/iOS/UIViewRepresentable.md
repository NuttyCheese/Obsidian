**`UIViewRepresentable`** — это протокол в **SwiftUI**, который позволяет **встраивать любой UIKit-компонент** (UIView или UIViewController) в SwiftUI-иерархию.

Это **единственный официальный и рекомендуемый** мост между UIKit и SwiftUI.

### Когда и зачем нужен UIViewRepresentable

| Ситуация                                      | Почему нужен именно UIViewRepresentable                  | Альтернатива (когда можно обойтись без него) |
|-----------------------------------------------|----------------------------------------------------------|----------------------------------------------|
| Использование `MKMapView`, `WKWebView`        | Нет нативного аналога в SwiftUI (Map в SwiftUI — упрощённый) | `Map` (только базовые функции)               |
| `UITextView`, `UITextField` с кастомным поведением | `TextEditor` / `TextField` ограничены                    | —                                            |
| `UIImagePickerController`, `MFMailComposeViewController` | Нет нативных аналогов                                    | —                                            |
| Кастомный `UIView` / `UIViewController` из старого проекта | Нужно перенести UIKit-код в SwiftUI без переписывания    | —                                            |
| `SCNView` (SceneKit), `ARSCNView` (ARKit)     | Полный контроль над 3D/AR-сценой                         | —                                            |
| Любые сложные UIKit-компоненты (AVPlayerViewController, PDFView и т.д.) | SwiftUI пока не покрывает всё                            | —                                            |

### Обязательные требования протокола UIViewRepresentable

```swift
protocol UIViewRepresentable : View {
    associatedtype UIViewType : UIView
    
    func makeUIView(context: Context) -> UIViewType
    func updateUIView(_ uiView: UIViewType, context: Context)
}
```

Опционально:
- `makeCoordinator() -> Coordinator` — если нужен делегат / состояние
- `dismantleUIView(_:context:)` — очистка при удалении (редко)
- `sizeThatFits(...)` — кастомный размер (iOS 16+)

### Самый популярный и рекомендуемый паттерн (2026 стандарт)

#### Пример: MKMapView в SwiftUI

```swift
import SwiftUI
import MapKit

struct MapView: UIViewRepresentable {
    
    @Binding var region: MKCoordinateRegion
    var annotations: [MKPointAnnotation]
    
    // Координатор — для делегата и состояния
    class Coordinator: NSObject, MKMapViewDelegate {
        var parent: MapView
        
        init(_ parent: MapView) {
            self.parent = parent
        }
        
        func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
            if annotation is MKUserLocation { return nil }
            
            let identifier = "CustomPin"
            var view = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) as? MKMarkerAnnotationView
            
            if view == nil {
                view = MKMarkerAnnotationView(annotation: annotation, reuseIdentifier: identifier)
                view?.canShowCallout = true
                view?.markerTintColor = .systemPurple
            } else {
                view?.annotation = annotation
            }
            
            return view
        }
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        mapView.showsUserLocation = true
        mapView.userTrackingMode = .follow
        
        return mapView
    }
    
    func updateUIView(_ uiView: MKMapView, context: Context) {
        // Обновляем регион
        uiView.setRegion(region, animated: true)
        
        // Обновляем аннотации
        uiView.removeAnnotations(uiView.annotations)
        uiView.addAnnotations(annotations)
    }
}

// Использование в SwiftUI
struct ContentView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173),
        span: MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)
    )
    
    let annotations = [
        MKPointAnnotation(coordinate: CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173), title: "Красная площадь")
    ]
    
    var body: some View {
        MapView(region: $region, annotations: annotations)
            .ignoresSafeArea()
    }
}
```

### Лучшие практики UIViewRepresentable в Swift 2026

- **Всегда** реализуйте `Coordinator`, если нужен делегат (MKMapViewDelegate, WKNavigationDelegate и т.д.)  
- **В `makeUIView`** — создавайте и настраивайте UIView один раз  
- **В `updateUIView`** — обновляйте состояние (регион, аннотации, контент, цвета и т.д.)  
- **Используйте** `@Binding` для двусторонней синхронизации (регион, выбранная аннотация)  
- **Для UIKit-делегатов** — делайте Coordinator наследником NSObject и реализуйте протоколы там  
- **Для производительности** — минимизируйте изменения в `updateUIView` — SwiftUI вызывает его часто  
- **Для AR / SceneKit** — `ARSCNView` и `SCNView` идеально встраиваются через UIViewRepresentable  
- **Документируйте** — пишите комментарий «UIViewRepresentable — обёртка над MKMapView с поддержкой аннотаций и региона»

**Короткий итог 2026**:
> `UIViewRepresentable` — это **мост** между UIKit и SwiftUI.  
> В 2026 году:  
> - обязателен для `MKMapView`, `WKWebView`, `UIImagePickerController`, `ARSCNView` и т.д.  
> - ключевые методы — `makeUIView`, `updateUIView`, `makeCoordinator`  
> - состояние передаётся через `@Binding` и `Coordinator`  
> - это **единственный** способ использовать мощные UIKit-компоненты в чистом SwiftUI-приложении  

Удачи с идеальной интеграцией UIKit-компонентов в SwiftUI! 🖼️↔️