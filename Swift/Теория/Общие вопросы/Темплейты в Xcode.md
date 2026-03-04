#xcode #cleanswift #swift #ios #templates #automation

---
## Введение

Если вы когда-нибудь устали создавать вручную одни и те же файлы (например, для VIPER, MVVM или просто для кастомного класса с определённой логикой), то **кастомные шаблоны [[Xcode]]** — это то, что вам нужно. Они позволяют автоматически генерировать файлы с нужной структурой, импортами и комментариями.

В этой статье мы разберем:
1.  Как устроены шаблоны Xcode.
2.  Как создать свой первый шаблон (пошагово).
3.  Где искать помощь и документацию, когда захочется большего.

---

## 1. Теория: как работают шаблоны

Xcode позволяет создавать два типа шаблонов:
- **File Templates** — для создания отдельных файлов (File -> New -> File).
- **Project Templates** — для создания целых проектов (File -> New -> Project).

Мы сосредоточимся на **File Templates**, так как они нужны чаще всего.

### 📁 Где хранятся шаблоны?
Шаблоны — это обычные папки с расширением **`.xctemplate`**, которые содержат файлы кода и конфигурацию .

- **Системные шаблоны Xcode** (лучше не трогать):  
  `/Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/` 
- **Пользовательские шаблоны** (наши):  
  `~/Library/Developer/Xcode/Templates/` 

Если папки `Templates` нет — создайте её самостоятельно.
Если вы не можете найти путь, возможно, они скрыты. Чтобы отобразить скрытые файлы или папки, надо нажать одновременно комбинацию клавиш -> **⌘ CMD + Shift + . (точка)**.

---

## 2. Создаем свой первый шаблон

Давай создадим простой шаблон для класса с протоколом (например, заготовка под модуль).

### Шаг 1: Создаем структуру папок
1.  Открой Finder.
2.  Нажми `Cmd+Shift+G` и перейди в `~/Library/Developer/Xcode/Templates/` .
3.  Создай папку (она станет категорией в меню Xcode), например, **My Templates**.
4.  Внутри создай папку с расширением `.xctemplate`, например: **`MVP Module.xctemplate`**.

    Структура будет выглядеть так:
    ```
    ~/Library/Developer/Xcode/Templates/
    └── My Templates/
        └── MVP Module.xctemplate/
    ```

### Шаг 2: Создаем TemplateInfo.plist
В папке `MVP Module.xctemplate` создай файл **`TemplateInfo.plist`**. Это самый важный файл — он говорит Xcode, как обрабатывать шаблон .

Вот минимальный рабочий пример:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Указываем, что это шаблон для подстановки текста -->
    <key>Kind</key>
    <string>Xcode.IDEKit.TextSubstitutionFileTemplateKind</string>
    
    <!-- Платформа (iOS) -->
    <key>Platforms</key>
    <array>
        <string>com.apple.platform.iphoneos</string>
    </array>
    
    <!-- Опции, которые видит пользователь при создании файла -->
    <key>Options</key>
    <array>
        <dict>
            <key>Identifier</key>
            <string>moduleName</string> <!-- Уникальный ID переменной -->
            <key>Required</key>
            <true/>
            <key>Name</key>
            <string>Module Name:</string> <!-- Подпись поля ввода -->
            <key>Description</key>
            <string>Enter the name of your module (e.g., Auth, Profile)</string>
            <key>Type</key>
            <string>text</string> <!-- Тип поля: текстовое поле -->
            <key>Default</key>
            <string>NewModule</string>
        </dict>
    </array>
</dict>
</plist>
```

> Подробнее про ключи: `Description`, `Summary`, `SortOrder`, `Platforms` можно найти в документации и примерах .

### Шаг 3: Создаем файлы шаблона
Теперь создадим сами файлы, которые будут генерироваться.

1.  В папке `MVP Module.xctemplate` создай файл **`___FILEBASENAME___Presenter.swift`**.
    - `___FILEBASENAME___` — это макрос, который заменится на имя, введенное пользователем на предыдущем шаге .
2.  Напиши в нем код:

```swift
//
//  ___FILENAME___
//  ___PROJECTNAME___
//
//  Created by ___FULLUSERNAME___ on ___DATE___.
//

import Foundation

// Используем переменную из TemplateInfo.plist
typealias ___VARIABLE_moduleName:identifier___ViewProtocol = AnyObject

// MARK: - Protocol
protocol ___VARIABLE_moduleName:identifier___PresenterProtocol {
    func viewDidLoad()
}

// MARK: - Presenter
final class ___FILEBASENAMEASIDENTIFIER___ {

    // MARK: - Properties
    private weak var view: ___VARIABLE_moduleName:identifier___ViewProtocol?
    private let router: ___VARIABLE_moduleName:identifier___Router

    // MARK: - Init
    init(view: ___VARIABLE_moduleName:identifier___ViewProtocol, router: ___VARIABLE_moduleName:identifier___Router) {
        self.view = view
        self.router = router
    }
}

// MARK: - PresenterProtocol
extension ___FILEBASENAMEASIDENTIFIER___: ___VARIABLE_moduleName:identifier___PresenterProtocol {
    func viewDidLoad() {
        // TODO: Setup initial state
    }
}
```

3.  Аналогично создай файл **`___FILEBASENAME___Router.swift`** с нужным содержимым.

**Важно:** `___VARIABLE_moduleName:identifier___` ссылается на переменную `moduleName`, которую мы объявили в `TemplateInfo.plist`. Пользователь введет имя, и оно подставится во все места .

### Шаг 4: Перезапускаем [[Xcode]]
Закрой Xcode (`Cmd+Q`) и открой заново. Теперь при создании нового файла (`Cmd+N`) в секции "My Templates" должен появиться наш шаблон "[[MVP (Model-View-Presenter) Architecture|MVP]] Module" .

---

## 3. Что можно использовать в шаблоне (макросы)

Внутри файлов шаблона можно использовать специальные макросы, которые Xcode заменяет на реальные значения .

| Макрос | Что заменяет |
| :--- | :--- |
| `___FILENAME___` | Полное имя файла (например, `MyClass.swift`). |
| `___FILEBASENAME___` | Имя файла без расширения (например, `MyClass`). |
| `___FILEBASENAMEASIDENTIFIER___` | Имя файла, пригодное для использования в качестве идентификатора в коде (например, `MyClass`). |
| `___PROJECTNAME___` | Название проекта. |
| `___FULLUSERNAME___` | Имя пользователя системы (для автора). |
| `___DATE___` | Текущая дата. |
| `___YEAR___` | Текущий год. |
| `___ORGANIZATIONNAME___` | Название организации (из настроек проекта). |
| `___VARIABLE_myVariable:identifier___` | Кастомная переменная `myVariable`, определенная в `TemplateInfo.plist`. |

---

## 4. Куда обращаться за помощью и ресурсы

Создание сложных шаблонов (с условиями, дропдаунами, чекбоксами, несколькими файлами) — это целое искусство. Вот где искать информацию и вдохновение:

1.  **Официальная документация Apple**
    - В первую очередь стоит смотреть **Apple Developer Documentation**. Хотя прямого раздела "Custom Templates" там немного, информация по структуре проекта и файлов может быть полезна.  (Managing files and folders)

2.  **Форумы Apple Developer**
    - Отличное место, чтобы понять, *куда копать*. Часто можно найти ответы от инженеров Apple или опытных разработчиков. 

3.  **Medium и блоги**
    - Много статей с пошаговыми гайдами. Особенно полезны те, где разбирают создание шаблонов для конкретных архитектур ([[VIPER Architecture|VIPER]], [[MVVM (Model-View-ViewModel) Architecture|MVVM]]). 

4.  **GitHub (главный источник вдохновения!)**
    - **Самый лучший способ изучить сложные вещи** — посмотреть, как это сделали другие. Ищите репозитории по запросу `xcode templates`.
    - Изучайте структуру папок и `TemplateInfo.plist` в чужих шаблонах.
    - Пример отличного шаблона для **[[Clean Swift (VIP) Architecture|CleanSwift]]**, который мы обсуждали ранее: [CleanSwift Templates](https://github.com/yonivav/CleanSwiftTemplates). Разберите его `plist` и вы поймете, как делать свои.

5.  **Технические энциклопедии и блоги (Tencent Cloud, CSDN, Zenn)**
    - Много информации на технических сайтах, где разработчики делятся примерами кода и конфигураций. 

---

## 5. Резюме

1.  **Где хранить:** `~/Library/Developer/Xcode/Templates/`.
2.  **Как называть папку:** `Имя.xctemplate`.
3.  **Главный файл:** `TemplateInfo.plist` (описывает переменные и поведение).
4.  **Как подставить имя:** `___FILEBASENAME___` в имени файла, `___VARIABLE_...___` внутри кода.
5.  **Где учиться:** Форум Apple, GitHub (чужие шаблоны), статьи на Medium.

Создание собственных шаблонов немного сложнее, чем просто скачать готовые. Но это дает вам **абсолютную гибкость**. Вы сможете создать идеальную заготовку для любого паттерна, принятого в вашей команде.