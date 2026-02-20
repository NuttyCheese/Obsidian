**`init(coder:)`** — это **обязательный инициализатор** в [[UIKit]]/[[AppKit]], который вызывается, когда объект ([[UIView]], [[UIViewController]], [[UITableViewCell]] и т.д.) создаётся из **Storyboard** или **.[[xib]]**-файла (через Interface Builder / nib).

Это один из самых важных и одновременно самых часто неправильно понимаемых методов в iOS-разработке.

### Когда и почему вызывается init(coder:)

| Ситуация                              | Вызывается init(coder:)? | Вызывается init(frame:)? | Что уже доступно в этот момент |
|---------------------------------------|---------------------------|---------------------------|---------------------------------|
| Загрузка из Storyboard / .xib         | **Да**                    | Нет                       | outlets и IBAction ещё nil      |
| Программное создание (UIView(frame:)) | Нет                       | **Да**                    | —                               |
| Программное создание через код        | Нет                       | **Да**                    | —                               |
| NSCoding / десериализация (редко)     | **Да**                    | Нет                       | —                               |

### Самые важные правила использования init(coder:) в 2026 году

1. **Обязательно вызывайте super.init(coder:)**  
   Если этого не сделать → runtime-краш при загрузке из Storyboard.

   ```swift
   required init?(coder: NSCoder) {
       super.init(coder: coder)
       // ваша инициализация
   }
   ```

2. **Нельзя** обращаться к **[[@IBOutlet]]** и **IBAction** внутри init(coder:)**  
   На момент вызова init(coder:) все outlets ещё **nil**.

   ```swift
   required init?(coder: NSCoder) {
       super.init(coder: coder)
       
       // Ошибка! avatarImageView будет nil
       avatarImageView.layer.cornerRadius = 20
   }
   ```

   **Правильное место** для работы с outlets — **[[awakeFromNib]]`()`**

   ```swift
   override func awakeFromNib() {
       super.awakeFromNib()
       
       // Теперь outlets уже не nil
       avatarImageView.layer.cornerRadius = avatarImageView.bounds.height / 2
       avatarImageView.clipsToBounds = true
   }
   ```

3. **Обычно** делают минимальную инициализацию

   ```swift
   class CustomView: UIView {
       
       // Обязательно required, если класс используется в Storyboard
       required init?(coder: NSCoder) {
           super.init(coder: coder)
           
           // Только то, что НЕ зависит от outlets и размеров
           backgroundColor = .clear
           layer.masksToBounds = false
       }
       
       // Для программного создания
       override init(frame: CGRect) {
           super.init(frame: frame)
           commonInit()
       }
       
       required init?(coder: NSCoder) {
           super.init(coder: coder)
           commonInit()
       }
       
       private func commonInit() {
           // общая инициализация для обоих init
       }
   }
   ```

### Самые частые ошибки с init(coder:) в 2026 году

| Ошибка                                              | Последствия                                      | Как исправить |
|-----------------------------------------------------|--------------------------------------------------|---------------|
| Забыли вызвать `super.init(coder:)`                 | Краш при загрузке из Storyboard                  | Всегда вызывать super |
| Обращение к IBOutlet внутри init(coder:)            | Fatal error: unexpectedly found nil              | Перенести в awakeFromNib() |
| Не сделали init(coder:) required                    | Ошибка компиляции при использовании в Storyboard | Добавить `required` |
| Делают сложную логику / сетевые запросы             | Задержка загрузки UI, возможные утечки           | Перенести в viewDidLoad / awakeFromNib |
| Используют init(coder:) в SwiftUI (UIViewRepresentable) | Метод вообще не вызывается                      | Не использовать — в SwiftUI другой жизненный цикл |

### Лучшие практики init(coder:) в 2026 году

- **Делайте** init(coder:) **required**, если класс используется в Storyboard / .xib
- **Вызывайте** `super.init(coder:)` **первой** строкой
- **Минимизируйте** логику внутри — только базовая настройка, не зависящая от outlets и размеров
- **Всю работу с IBOutlet** — переносите в `awakeFromNib()`
- **Всю работу с layout / frame / bounds** — переносите в `layoutSubviews()` или `didMoveToSuperview()`
- **В SwiftUI** — `init(coder:)` **не вызывается** (используйте `makeUIView` / `updateUIView` в UIViewRepresentable)
- **Документируйте** — пишите комментарий «required init?(coder:) — инициализация из Storyboard / .xib»

**Короткий итог 2026**:
> `init(coder:)` — это **инициализатор для объектов, загруженных из nib/Storyboard**.  
> В 2026 году:  
> - **обязательно** вызывайте `super.init(coder:)`  
> - **нельзя** обращаться к IBOutlet внутри него (они ещё nil)  
> - **минимальная** логика: базовая настройка, не зависящая от outlets  
> - всё остальное — в `awakeFromNib()`, `viewDidLoad()`, `layoutSubviews()`  
> Это **классический** и **очень важный** метод при работе с Interface Builder.
