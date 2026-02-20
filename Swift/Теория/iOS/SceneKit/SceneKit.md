**SceneKit** — это высокоуровневый 3D-фреймворк от Apple, предназначенный для создания и рендеринга 3D-сцен в реальном времени.

Он идеально подходит для:
- AR-приложений (вместе с ARKit),
- игр (особенно простых и средних по сложности),
- визуализаций (3D-модели продуктов, интерьеры, научные данные),
- интерактивных 3D-интерфейсов (просмотр моделей, кастомные элементы управления).

SceneKit работает **на GPU** и использует **physically-based rendering** (PBR), что даёт очень качественную картинку с минимальными усилиями.

### Основные сущности SceneKit (2026 актуально)

| Компонент         | Класс / Протокол                    | Что это такое                                                | Самый частый сценарий             |
| ----------------- | ----------------------------------- | ------------------------------------------------------------ | --------------------------------- |
| **Сцена**         | `SCNScene`                          | Контейнер для всей 3D-сцены (свет, камера, объекты)          | Загрузка .scn / .dae / .usdz      |
| **View**          | `SCNView`                           | UIView для отображения сцены (аналог GLKView)                | Основной контейнер в [[UIKit]]    |
| **Узел**          | `SCNNode`                           | Базовый строительный блок (объект, свет, камера)             | Всё в сцене — это узлы            |
| **Геометрия**     | `SCNGeometry`                       | Форма объекта (куб, сфера, плоскость, кастомная)             | `SCNBox`, `SCNSphere`, `SCNPlane` |
| **Материал**      | `SCNMaterial`                       | Внешний вид поверхности (PBR: diffuse, metalness, roughness) | Реалистичная текстура             |
| **Камера**        | `SCNCamera`                         | Точка обзора (perspective / orthographic)                    | Управление зумом, FOV             |
| **Свет**          | `SCNLight`                          | Источники света (omni, directional, spot, IES)               | Освещение сцены                   |
| **Анимация**      | `SCNAction`, `SCNAnimationPlayer`   | Простые действия и сложные анимации из .dae/.usdz            | Движение, вращение, fade          |
| **Физика**        | `SCNPhysicsBody`, `SCNPhysicsWorld` | Столкновения, гравитация, силы                               | Падение объектов, ragdoll         |
| **Частицы**       | `SCNParticleSystem`                 | Системы частиц (дым, огонь, искры, дождь)                    | Эффекты                           |
| **AR-интеграция** | `ARSCNView`                         | SCNView + ARSession (ARKit)                                  | AR-приложения                     |

### Минимальный рабочий пример ([[UIKit]] + SceneKit)

```swift
import UIKit
import SceneKit

@MainActor
class SceneViewController: UIViewController {
    
    private var sceneView: SCNView!
    
    override func loadView() {
        sceneView = SCNView(frame: .zero)
        sceneView.allowsCameraControl = true          // встроенное управление (pan, pinch, rotate)
        sceneView.autoenablesDefaultLighting = true   // автоматический свет
        sceneView.backgroundColor = .black
        sceneView.showsStatistics = true              // FPS, узлы, полигоны (для отладки)
        
        view = sceneView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаём сцену
        let scene = SCNScene()
        sceneView.scene = scene
        
        // Добавляем камеру (по умолчанию есть, но можно настроить)
        let cameraNode = SCNNode()
        cameraNode.camera = SCNCamera()
        cameraNode.position = SCNVector3(x: 0, y: 0, z: 10)
        scene.rootNode.addChildNode(cameraNode)
        
        // Добавляем объект — куб
        let cube = SCNBox(width: 2, height: 2, length: 2, chamferRadius: 0.2)
        let material = SCNMaterial()
        material.diffuse.contents = UIColor.systemBlue
        material.metalness.contents = 0.8
        material.roughness.contents = 0.2
        cube.materials = [material]
        
        let cubeNode = SCNNode(geometry: cube)
        cubeNode.position = SCNVector3(0, 0, 0)
        scene.rootNode.addChildNode(cubeNode)
        
        // Добавляем вращение
        let rotationAction = SCNAction.rotateBy(x: 0, y: CGFloat.pi * 2, z: 0, duration: 10)
        cubeNode.runAction(SCNAction.repeatForever(rotationAction))
        
        // Добавляем свет
        let lightNode = SCNNode()
        lightNode.light = SCNLight()
        lightNode.light?.type = .omni
        lightNode.position = SCNVector3(x: 0, y: 10, z: 10)
        scene.rootNode.addChildNode(lightNode)
    }
}
```

### Лучшие практики SceneKit в Swift 2026

- **Всегда** используйте `SCNView` в UIKit или `SceneView` в [[SwiftUI]]  
- **Загружайте модели** в форматах `.usdz`, `.scn`, `.dae` (Collada) — через `SCNScene(named:)` или `SCNScene(url:)`  
- **Для реализма** — используйте **PBR-материалы** (`metalness`, `roughness`, `normal`)  
- **Для анимаций** — предпочитайте `SCNAction` для простых эффектов и `CAAnimation` / `SCNAnimationPlayer` для сложных  
- **Для физики** — включайте `scene.physicsWorld` и задавайте `SCNPhysicsBody` узлам  
- **Для AR** — используйте `ARSCNView` + `ARSession` (ARKit)  
- **Для производительности** — минимизируйте количество полигонов, используйте LOD (уровни детализации), отключайте ненужные тени/отражения  
- **Privacy Manifest** — обязателен с iOS 17+ при использовании SceneKit + ARKit  
- **Документируйте** — пишите комментарий «SCNScene — 3D-модель куба с PBR-материалом и вращением»

**Короткий итог 2026**:
> SceneKit — это **высокоуровневый 3D-движок** Apple для создания сцен, моделей, анимаций и AR.  
> В 2026 году:  
> - основной view — `SCNView` (UIKit) / `SceneView` (SwiftUI)  
> - всё строится из `SCNNode` + `SCNGeometry` + `SCNMaterial`  
> - анимации — `SCNAction` (простые) и `SCNAnimationPlayer` (сложные)  
> - физика и частицы — встроенные и очень мощные  
> Это **самый простой** и **самый производительный** способ добавить 3D в iOS-приложение без Unity/Unreal.
