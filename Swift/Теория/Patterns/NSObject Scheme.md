# Схема иерархии классов UIKit (актуальная на 2026 год)

Вот актуальная (на февраль 2026 года) **таблица** основных классов UIKit с их цепочкой наследования.  
Я сделал её в формате Markdown-таблицы — удобно копировать в Obsidian, Notion или любой Markdown-редактор.  

Таблица охватывает самые используемые и ключевые классы (более 80).  
Базовая структура осталась неизменной с iOS 13–18, новые классы 2024–2026 годов (типа Liquid Glass-related, UIBackgroundExtensionView и т.п.) добавлены где они логично вписываются, но радикальных перестроек иерархии не было.

| №  | Класс                              | Прямой суперкласс              | Цепочка наследования (кратко)                                      | Комментарий / Когда используется                          |
|----|------------------------------------|--------------------------------|--------------------------------------------------------------------|-----------------------------------------------------------|
| 1  | NSObject                           | —                              | NSObject                                                           | Базовый класс всего                                        |
| 2  | UIResponder                        | NSObject                       | NSObject → UIResponder                                             | Обработка событий, first responder                         |
| 3  | UIView                             | UIResponder                    | NSObject → UIResponder → UIView                                    | Базовый визуальный элемент                                 |
| 4  | UIControl                          | UIView                         | … → UIView → UIControl                                             | Базовый класс для интерактивных контролов                  |
| 5  | UIButton                           | UIControl                      | … → UIView → UIControl → UIButton                                  | Кнопка (самый частый пример)                               |
| 6  | UISwitch                           | UIControl                      | … → UIView → UIControl → UISwitch                                  | Переключатель                                              |
| 7  | UISlider                           | UIControl                      | … → UIView → UIControl → UISlider                                  | Слайдер                                                    |
| 8  | UIStepper                          | UIControl                      | … → UIView → UIControl → UIStepper                                 | Степпер (+/-)                                              |
| 9  | UISegmentedControl                 | UIControl                      | … → UIView → UIControl → UISegmentedControl                        | Сегментированный контрол                                   |
| 10 | UIPageControl                      | UIControl                      | … → UIView → UIControl → UIPageControl                             | Точки страниц                                              |
| 11 | UIDatePicker                       | UIControl                      | … → UIView → UIControl → UIDatePicker                              | Выбор даты/времени                                         |
| 12 | UIPickerView                       | UIView                         | … → UIView → UIPickerView                                          | Колёсико выбора (не наследует UIControl)                   |
| 13 | UITextField                        | UIControl                      | … → UIView → UIControl → UITextField                               | Поле ввода одной строки                                    |
| 14 | UITextView                         | UIScrollView                   | … → UIView → UIScrollView → UITextView                             | Многострочный текст                                        |
| 15 | UIScrollView                       | UIView                         | … → UIView → UIScrollView                                          | Скроллируемая область                                      |
| 16 | UITableView                        | UIScrollView                   | … → UIView → UIScrollView → UITableView                            | Таблица (список)                                           |
| 17 | UICollectionView                   | UIScrollView                   | … → UIView → UIScrollView → UICollectionView                       | Коллекция / кастомные layouts                              |
| 18 | UIImageView                        | UIView                         | … → UIView → UIImageView                                           | Отображение картинок                                       |
| 19 | UILabel                            | UIView                         | … → UIView → UILabel                                               | Текст (однострочный / многострочный)                       |
| 20 | UIActivityIndicatorView            | UIView                         | … → UIView → UIActivityIndicatorView                               | Спиннер загрузки                                           |
| 21 | UIProgressView                     | UIView                         | … → UIView → UIProgressView                                        | Прогресс-бар                                               |
| 22 | UIVisualEffectView                 | UIView                         | … → UIView → UIVisualEffectView                                    | Blur / vibrancy                                            |
| 23 | UIStackView                        | UIView                         | … → UIView → UIStackView                                           | Автоматический стек (H/V)                                  |
| 24 | UINavigationBar                    | UIView                         | … → UIView → UINavigationBar                                       | Навбар                                                     |
| 25 | UIToolbar                          | UIView                         | … → UIView → UIToolbar                                             | Тулбар                                                     |
| 26 | UITabBar                           | UIView                         | … → UIView → UITabBar                                              | Нижний таб-бар                                             |
| 27 | UISearchBar                        | UIView                         | … → UIView → UISearchBar                                           | Поисковая строка                                           |
| 28 | UIWindow                           | UIView                         | NSObject → UIResponder → UIView → UIWindow                         | Окно приложения                                            |
| 29 | UIViewController                   | UIResponder                    | NSObject → UIResponder → UIViewController                          | Контроллер экрана                                          |
| 30 | UINavigationController             | UIViewController               | … → UIViewController → UINavigationController                      | Навигация (push/pop)                                       |
| 31 | UITabBarController                 | UIViewController               | … → UIViewController → UITabBarController                          | Табы (вкладки)                                             |
| 32 | UISplitViewController              | UIViewController               | … → UIViewController → UISplitViewController                       | Split-screen (iPad)                                        |
| 33 | UIPageViewController               | UIViewController               | … → UIViewController → UIPageViewController                        | Страничный скролл (как книги)                              |
| 34 | UITableViewController              | UIViewController               | … → UIViewController → UITableViewController                       | Контроллер + готовая таблица                               |
| 35 | UICollectionViewController         | UIViewController               | … → UIViewController → UICollectionViewController                  | Контроллер + коллекция                                     |
| 36 | UIAlertController                  | UIViewController               | … → UIViewController → UIAlertController                           | Алерт / экшн-шит                                           |
| 37 | UIActivityViewController           | UIViewController               | … → UIViewController → UIActivityViewController                    | Share-sheet                                                |
| 38 | UIImagePickerController            | UINavigationController         | … → UINavigationController → UIImagePickerController               | Камера / галерея                                           |
| 39 | UIColorPickerViewController        | UIViewController               | … → UIViewController → UIColorPickerViewController                 | Выбор цвета                                                |
| 40 | UIFontPickerViewController         | UIViewController               | … → UIViewController → UIFontPickerViewController                  | Выбор шрифта                                               |
| 41 | UIDocumentPickerViewController     | UIViewController               | … → UIViewController → UIDocumentPickerViewController              | Выбор файлов                                               |
| 42 | UIHostingController                | UIViewController               | … → UIViewController → UIHostingController<Content: View>          | SwiftUI внутри UIKit                                       |
| 43 | UIApplication                      | UIResponder                    | NSObject → UIResponder → UIApplication                             | Главный объект приложения                                  |
| 44 | UIBarButtonItem                    | UIBarItem                      | NSObject → UIBarItem → UIBarButtonItem                             | Кнопка в навбаре / тулбаре                                 |
| 45 | UIBarItem                          | NSObject                       | NSObject → UIBarItem                                               | Базовый для бар-айтемов                                    |
| 46 | UIMenu                             | NSObject                       | NSObject → UIMenu                                                  | Контекстное меню (новое)                                   |
| 47 | UIAction                           | NSObject                       | NSObject → UIAction                                                | Действие для кнопок/меню                                   |
| 48 | UIGestureRecognizer                | NSObject                       | NSObject → UIGestureRecognizer                                     | Базовый распознаватель жестов                              |
| 49 | UIPanGestureRecognizer             | UIGestureRecognizer            | … → UIGestureRecognizer → UIPanGestureRecognizer                   | Перетаскивание                                             |
| 50 | UITapGestureRecognizer             | UIGestureRecognizer            | … → UIGestureRecognizer → UITapGestureRecognizer                   | Тап                                                        |

### Дополнительные важные классы (2024–2026)

| №  | Класс                              | Прямой суперкласс | Цепочка кратко                          | Примечание (новое / актуальное в 2026)                     |
|----|------------------------------------|-------------------|-----------------------------------------|-------------------------------------------------------------|
| 51 | UIBackgroundExtensionView          | UIView            | … → UIView → UIBackgroundExtensionView  | Расширение контента в unsafe areas (iOS 19/26)              |
| 52 | UIButton.Configuration             | —                 | (не класс-наследник, конфиг)            | Liquid Glass стили: glass(), prominentGlass() и т.д.       |
| 53 | UIViewController.Transition        | —                 | (новый тип)                             | zoom с sourceBarButtonItemProvider                          |

