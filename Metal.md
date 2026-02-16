**Metal** — это низкоуровневый графический и вычислительный [[API]] от Apple, который позволяет максимально эффективно использовать GPU (графический процессор) на всех платформах Apple: [[iOS]], iPadOS, macOS, tvOS, visionOS и watchOS (ограниченно).

По состоянию на февраль 2026 года Metal остаётся **единственным современным способом** получить **максимальную производительность** от GPU Apple Silicon (M- и A-серии чипов), особенно в задачах, требующих:

- высокопроизводительного рендеринга 3D/2D графики  
- вычислений на GPU (GPGPU / compute shaders)  
- машинного обучения (нейросети inference)  
- обработки видео/изображений в реальном времени  
- AR/VR/пространственного рендеринга

### Ключевые возможности Metal в 2026 году (Metal 3+)

| Возможность                          | Описание / Классы                              | Минимальная версия ОС | Когда использовать в 2026 | Производительность (примерно) |
|--------------------------------------|------------------------------------------------|------------------------|----------------------------|-------------------------------|
| **Metal 3** (2022–2026)              | Ray Tracing (аппаратный), Mesh Shaders, Dynamic Cache | iOS 16+, macOS 13+     | Современные игры, AR/VR, визуализация | ★★★★★ (лидер)                 |
| **MetalFX Upscaling**                | Пространственное и временное апскейлинг (аналог DLSS/FSR) | iOS 16+, macOS 13+     | Игры, высокое разрешение на слабом GPU | ★★★★★                         |
| **Mesh Shaders**                     | Процессинг геометрии на GPU (тесселяция, LOD) | iOS 15+, macOS 12+     | Высокодетализированные модели, procedural geometry | ★★★★★                         |
| **Ray Tracing (аппаратный)**         | Трассировка лучей на GPU (A17 Pro, M3+)       | iOS 16+, macOS 13+     | Реалистичное освещение, отражения, тени | ★★★★★ (только новые чипы)    |
| **Metal Compute Shaders**            | GPGPU — вычисления на GPU                      | iOS 8+, macOS 10.11+   | ML inference, обработка изображений, физика | ★★★★★                         |
| **Argument Buffers / Indirect Command Buffers** | Динамическая подача данных и команд на GPU | iOS 11+, macOS 10.13+  | Высокая плотность draw calls, большие сцены | ★★★★★                         |
| **Metal Performance HUD**            | Встроенный оверлей для профилирования FPS, GPU/CPU usage | iOS 15+, macOS 12+     | Отладка в реальном времени | ★★★★★                         |
| **Metal Developer Tools for Windows** | Metal на Windows (для разработки)             | —                      | Разработка кросс-платформенных игр | ★★★★☆                         |

### Самый популярный и рекомендуемый паттерн 2026 — MTKView + Metal compute/render pipeline

```swift
import MetalKit

class MetalRenderer: NSObject, MTKViewDelegate {
    
    private let device: MTLDevice
    private let commandQueue: MTLCommandQueue
    private let pipelineState: MTLRenderPipelineState
    
    init?(mtkView: MTKView) {
        guard let device = MTLCreateSystemDefaultDevice(),
              let commandQueue = device.makeCommandQueue() else {
            return nil
        }
        
        self.device = device
        self.commandQueue = commandQueue
        
        mtkView.device = device
        mtkView.delegate = self
        
        // Создаём простой шейдер (vertex + fragment)
        let library = device.makeDefaultLibrary()
        let descriptor = MTLRenderPipelineDescriptor()
        descriptor.vertexFunction = library?.makeFunction(name: "vertexShader")
        descriptor.fragmentFunction = library?.makeFunction(name: "fragmentShader")
        descriptor.colorAttachments[0].pixelFormat = mtkView.colorPixelFormat
        
        pipelineState = try! device.makeRenderPipelineState(descriptor: descriptor)
    }
    
    func draw(in view: MTKView) {
        guard let commandBuffer = commandQueue.makeCommandBuffer(),
              let renderPassDescriptor = view.currentRenderPassDescriptor,
              let renderEncoder = commandBuffer.makeRenderCommandEncoder(descriptor: renderPassDescriptor) else {
            return
        }
        
        renderEncoder.setRenderPipelineState(pipelineState)
        renderEncoder.drawPrimitives(type: .triangle, vertexStart: 0, vertexCount: 3)
        renderEncoder.endEncoding()
        
        if let drawable = view.currentDrawable {
            commandBuffer.present(drawable)
        }
        
        commandBuffer.commit()
    }
    
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {
        // Обновляем viewport при смене размера
    }
}
```

### Лучшие практики Metal в Swift 2026

- **MTKView** — основной способ отображения Metal-контента в UIKit/AppKit  
- **Metal 3 features** — используй на A17 Pro/M3+ (ray tracing, mesh shaders, MetalFX)  
- **Compute shaders** — для ML inference, обработки изображений, физики — самый частый сценарий  
- **Argument Buffers** — для больших сцен (тысячи draw calls)  
- **@MainActor** — MTKView и UI — на главном акторе  
- **Swift 6 strict concurrency** — все Metal-объекты Sendable-safe, но буферы и текстуры — careful с передачей между задачами  
- **Профилирование** — Metal HUD + Instruments (GPU Counters, Metal System Trace)  
- **Документируйте** — пиши комментарий «Metal 3 compute shader — обработка текстур на GPU»

**Короткий девиз 2026**:
> «Metal — это когда тебе нужна максимальная производительность GPU на устройствах Apple.  
> В 2026 году это **единственный выбор** для серьёзной 3D-графики, ray tracing, кастомного рендеринга и GPGPU.  
> Для простых анимаций и UI — SwiftUI / UIKit / SceneKit / RealityKit.»

Удачи с высокопроизводительной графикой и вычислениями на GPU в Swift! ⚙️