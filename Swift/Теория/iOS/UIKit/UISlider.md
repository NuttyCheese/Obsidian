**UISlider** — это классический элемент управления в [[UIKit]], который позволяет пользователю **плавно выбирать значение** из заданного диапазона с помощью ползунка (слайдера).

В 2026 году UISlider остаётся **одним из самых надёжных и часто используемых** контроллеров для настройки числовых параметров: громкость, яркость, размер шрифта, интенсивность эффекта, скорость воспроизведения и т.д.

### Когда использовать UISlider в 2026 году (реальные кейсы)

| Сценарий                              | Почему именно UISlider (а не Stepper / Picker)       | Альтернатива в SwiftUI |
|---------------------------------------|-------------------------------------------------------|-------------------------|
| Настройка громкости / яркости         | Плавное, непрерывное изменение                        | `Slider`                |
| Выбор значения в большом диапазоне    | Удобно двигать пальцем на большой дистанции           | `Slider`                |
| Регулировка параметров в реальном времени | Значение меняется мгновенно при движении ползунка     | `Slider`                |
| Индикация прогресса + пользовательский ввод | Можно сделать readonly + интерактивный                | `Slider` + `.disabled()` |
| Настройка с точностью до 0.1–1.0      | Легко задать шаг через округление в обработчике       | `Slider` с `step`       |

### Сравнение UISlider vs другие контроллеры ввода числа

| Контроллер           | Диапазон      | Шаг изменения             | Непрерывность | Лучше всего для                       | Минусы                           |
| -------------------- | ------------- | ------------------------- | ------------- | ------------------------------------- | -------------------------------- |
| **UISlider**         | любой         | любой (через округление)  | Да            | плавная регулировка, большой диапазон | нет точного ввода                |
| **[[UIStepper]]**    | любой         | фиксированный (stepValue) | Нет           | точное пошаговое изменение            | маленький диапазон               |
| **[[UITextField]]**  | любой         | любой                     | Нет           | точный ввод числа с клавиатуры        | неудобно для плавной регулировки |
| **[[UIPickerView]]** | фиксированный | фиксированный             | Нет           | выбор из списка                       | не для непрерывного диапазона    |

### Самый популярный и рекомендуемый паттерн 2026 года  
(UISlider + [[Auto Layout]] + [[Combine]] + динамическое обновление UI)

```swift
import UIKit
import Combine

class VolumeControlViewController: UIViewController {
    
    private let viewModel = VolumeViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var slider: UISlider = {
        let s = UISlider()
        s.minimumValue = 0
        s.maximumValue = 1
        s.value = Float(viewModel.volume)
        s.minimumTrackTintColor = .systemBlue
        s.maximumTrackTintColor = .systemGray5
        s.thumbTintColor = .systemBlue
        s.isContinuous = true  // обновление в реальном времени
        s.addTarget(self, action: #selector(sliderValueChanged), for: .valueChanged)
        s.translatesAutoresizingMaskIntoConstraints = false
        return s
    }()
    
    private lazy var valueLabel: UILabel = {
        let lbl = UILabel()
        lbl.textAlignment = .center
        lbl.font = .systemFont(ofSize: 32, weight: .bold)
        lbl.text = String(format: "%.0f%%", viewModel.volume * 100)
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    private lazy var iconImageView: UIImageView = {
        let iv = UIImageView(image: UIImage(systemName: "speaker.wave.3.fill"))
        iv.contentMode = .scaleAspectFit
        iv.tintColor = .systemBlue
        iv.translatesAutoresizingMaskIntoConstraints = false
        return iv
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Громкость"
        view.backgroundColor = .systemBackground
        
        let stack = UIStackView(arrangedSubviews: [iconImageView, slider, valueLabel])
        stack.axis = .horizontal
        stack.spacing = 16
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 32),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -32)
        ])
        
        bindViewModel()
    }
    
    private func bindViewModel() {
        // Slider → ViewModel
        slider.publisher(for: \.value)
            .map { Double($0) }
            .removeDuplicates()
            .assign(to: \.volume, on: viewModel)
            .store(in: &cancellables)
        
        // ViewModel → Label и иконка
        viewModel.$volume
            .map { String(format: "%.0f%%", $0 * 100) }
            .assign(to: \.text, on: valueLabel)
            .store(in: &cancellables)
        
        viewModel.$volume
            .map { $0 > 0.7 ? "speaker.wave.3.fill" : "speaker.wave.2.fill" }
            .map { UIImage(systemName: $0) }
            .assign(to: \.image, on: iconImageView)
            .store(in: &cancellables)
    }
    
    @objc private func sliderValueChanged(_ sender: UISlider) {
        // Можно добавить лёгкую вибрацию при изменении
        let generator = UIImpactFeedbackGenerator(style: .light)
        generator.impactOccurred()
    }
}

// ViewModel
@MainActor
class VolumeViewModel: ObservableObject {
    @Published var volume: Double = 0.5
}
```

### Лучшие практики UISlider в 2026 году

- **Всегда** задавайте `minimumValue` и `maximumValue` — без них ползунок не работает  
- **Для целых значений** — округляйте в обработчике: `let value = Int(round(sender.value))`  
- **Для непрерывного обновления** — оставляйте `isContinuous = true` (по умолчанию)  
- **Для визуальной обратной связи** — меняйте `thumbTintColor` или добавляйте вибрацию при достижении предела  
- **Для Combine** — используйте `.publisher(for: \.value)` — это самый удобный способ двусторонней привязки  
- **Для доступности** — задавайте `accessibilityLabel = "Громкость"` и `accessibilityValue = "\(Int(value * 100))%"`  
- **Для SwiftUI** — используйте `Slider` — UISlider нужен только в UIKit или смешанных проектах  
- **Документируйте** — пишите комментарий:

```swift
/// UISlider для регулировки громкости (0–100%)
private lazy var volumeSlider: UISlider = {
    let s = UISlider()
    s.minimumValue = 0
    s.maximumValue = 1
    s.value = Float(viewModel.volume)
    s.minimumTrackTintColor = .systemBlue
    return s
}()
```

**Короткий итог 2026**:
> `UISlider` — **ползунок** для плавного выбора значения из диапазона.  
> В 2026 году:  
> - ключевые свойства — `value`, `minimumValue`, `maximumValue`, `isContinuous`  
> - самый популярный паттерн — двусторонняя привязка через Combine + визуальная обратная связь  
> - идеален для громкости, яркости, прозрачности, интенсивности, прогресса  
> - в SwiftUI — заменяется на `Slider`  
> - это **надёжный** и **универсальный** контроллер, который до сих пор используется почти в каждом приложении  
