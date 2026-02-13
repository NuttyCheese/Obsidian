**Accelerate** — это мощный фреймворк Apple для высокопроизводительных вычислений на [[iOS]], macOS, watchOS и tvOS. Он предоставляет оптимизированные функции для работы с векторами, матрицами, цифровой обработкой сигналов (DSP), линейной алгеброй, быстрым преобразованием Фурье (FFT) и многим другим.

По состоянию на февраль 2026 года Accelerate остаётся **одним из самых быстрых и энергоэффективных** способов выполнять тяжёлые математические вычисления на устройствах Apple (особенно на чипах M- и A-серии).

### Основные подмодули Accelerate (2026 актуальные)

| Подмодуль / Фреймворк       | Что делает лучше всего                              | Самые популярные функции 2026 | Когда использовать |
|------------------------------|------------------------------------------------------|--------------------------------|---------------------|
| **vDSP** (Vector Digital Signal Processing) | Быстрые операции над массивами (FFT, фильтры, свёртка) | `vDSP_fft_zip`, `vDSP_conv`, `vDSP_vmul` | Аудио, обработка сигналов, ML-preprocessing |
| **BLAS** (Basic Linear Algebra Subprograms) | Векторные и матричные операции (умножение, сложение) | `cblas_sgemm`, `cblas_sdot` | Линейная алгебра, нейросети |
| **LAPACK**                   | Решение систем уравнений, разложение матриц          | `sgesv`, `sgeev`               | Научные вычисления, физика |
| **BNNS** (Basic Neural Network Subroutines) | Низкоуровневые операции для нейросетей (conv, matmul) | `BNNSConvolutionLayer`, `BNNSMatrixMultiplication` | Лёгкие on-device модели |
| **Accelerate** (общий)       | Всё выше + векторные функции, статистика, квантизация | `vDSP_vfill`, `vDSP_vsqr`      | Универсальный доступ |

### Самые популярные сценарии использования Accelerate в iOS-приложениях 2026

1. **Обработка аудио в реальном времени**  
   → FFT для спектрального анализа, фильтры, эквалайзер

2. **Компьютерное зрение / обработка изображений**  
   → Свёртка, фильтрация, resize, цветокоррекция

3. **Лёгкие on-device ML-модели**  
   → Предобработка (нормализация, resize), постобработка, inference на BNNS

4. **Игры и AR**  
   → Матричные преобразования, физика частиц, векторная математика

5. **Научные приложения**  
   → Статистика, фильтрация сигналов, решение СЛАУ

### Пример №1 — Быстрый FFT (самый частый сценарий)

```swift
import Accelerate

func computeFFT(signal: [Float]) -> [Float] {
    let n = signal.count
    guard n.isPowerOf2 else { fatalError("Signal length must be power of 2") }
    
    // 1. Подготовка комплексного буфера
    var real = signal
    var imag = [Float](repeating: 0, count: n)
    
    var splitComplex = DSPSplitComplex(realp: &real, imagp: &imag)
    
    // 2. Создание плана FFT
    let log2n = vDSP_Length(log2(Float(n)))
    guard let fftSetup = vDSP_create_fftsetup(log2n, FFTRadix(kFFTRadix2)) else {
        fatalError("Failed to create FFT setup")
    }
    defer { vDSP_destroy_fftsetup(fftSetup) }
    
    // 3. Прямое FFT (in-place)
    vDSP_fft_zip(fftSetup, &splitComplex, 1, log2n, FFTDirection(FFT_FORWARD))
    
    // 4. Получение амплитуд (magnitude)
    var magnitudes = [Float](repeating: 0, count: n/2)
    vDSP_zvmags(&splitComplex, 1, &magnitudes, 1, vDSP_Length(n/2))
    
    // 5. Нормализация (часто нужно)
    var scale = 1.0 / Float(n)
    vDSP_vsmul(magnitudes, 1, &scale, &magnitudes, 1, vDSP_Length(n/2))
    
    return magnitudes
}
```

### Пример №2 — Матричное умножение (BNNS / BLAS)

```swift
import Accelerate

func matrixMultiply(A: [Float], B: [Float], m: Int, n: Int, k: Int) -> [Float] {
    var C = [Float](repeating: 0, count: m * n)
    
    cblas_sgemm(
        CblasRowMajor,     // порядок хранения
        CblasNoTrans,      // A не транспонирована
        CblasNoTrans,      // B не транспонирована
        CblasInt(m),       // rows A
        CblasInt(n),       // cols B
        CblasInt(k),       // cols A / rows B
        1.0,               // alpha
        A,                 // A
        CblasInt(k),       // lda
        B,                 // B
        CblasInt(n),       // ldb
        0.0,               // beta
        &C,                // C
        CblasInt(n)        // ldc
    )
    
    return C
}
```

### Сравнение Accelerate с альтернативами 2026

| Библиотека / Фреймворк     | Скорость | Легкость использования | Поддержка Apple Silicon | Поддержка Swift Concurrency | Рекомендация 2026 |
|-----------------------------|----------|--------------------------|--------------------------|------------------------------|-------------------|
| **Accelerate**              | ★★★★★    | ★★★★☆                    | ★★★★★                    | ★★★★★                        | Основной выбор    |
| **Metal Performance Shaders** (MPS) | ★★★★★    | ★★★☆☆                    | ★★★★★                    | ★★★★☆                        | GPU-ускорение     |
| **Core ML**                 | ★★★★☆    | ★★★★★                    | ★★★★★                    | ★★★★★                        | Готовые модели    |
| **BNNS** (внутри Accelerate)| ★★★★★    | ★★★☆☆                    | ★★★★★                    | ★★★★★                        | Лёгкие сети       |
| **vDSP / BLAS / LAPACK**    | ★★★★★    | ★★★★☆                    | ★★★★★                    | ★★★★★                        | Чистая математика |

### Лучшие практики Accelerate в Swift 2026

- **Всегда проверяй длину массива** — многие функции требуют power-of-2  
- **Используй `DSPSplitComplex` для FFT** — это самый быстрый путь  
- **Не забывай про `defer { vDSP_destroy_fftsetup(...) }`**  
- **Для SwiftUI / async** — оборачивай в `Task.detached` или `actor`  
- **Профилируй в Instruments** — Accelerate обычно быстрее, чем ручной код, но проверяй  
- **Swift 6 strict concurrency** — Accelerate-функции thread-safe, но массивы должны быть Sendable  
- **Документируйте** — пиши комментарий «Accelerate vDSP — FFT для спектрального анализа»

**Короткий девиз 2026**:
> «Accelerate — это когда тебе нужно выжать максимум производительности из CPU/GPU без боли и компромиссов.  
> В 2026 году это **лучший выбор** для DSP, линейной алгебры и лёгких нейросетей на устройстве.»
