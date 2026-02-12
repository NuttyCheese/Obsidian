#crash #warning #xcode #swift
### Что это значит

[[Xcode]] не может найти указанный ресурс (изображение, xib, storyboard, JSON, шрифт и т.д.), который должен быть включён в **Bundle** приложения.

Чаще всего это предупреждение возникает, когда:

1. Ресурс был удалён или переименован в проекте.
    
2. Ресурс не добавлен в **Target Membership** (не принадлежит целевому таргету).
    
3. Ресурс находится в неподходящей папке и не включается в сборку.
    
4. Используется неправильный путь при обращении к ресурсу в коде.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: [[UIImage]]**

```swift
let image = UIImage(named: "logo.png")
```

- Если `logo.png` удалён или не включён в Target → Xcode выдаёт предупреждение:
    

```
Resource logo.png not found
```

---

**Пример 2: Bundle [[JSON]]**

```swift
guard let url = Bundle.main.url(forResource: "config", withExtension: "json") else { return }
let data = try Data(contentsOf: url)
```

- Если `config.json` отсутствует в сборке → предупреждение `Resource config.json not found`.
    

---

### Как исправить

#### 1️⃣ Проверка наличия файла

- Убедитесь, что файл существует в проекте (Project Navigator).
    

#### 2️⃣ Target Membership

- Выберите файл → правый панель Inspector → **Target Membership** → отметьте ваш таргет.
    

#### 3️⃣ Проверка пути

- Для ресурсов в подпапках можно использовать **Folder Reference** (`blue folder`) или корректный путь в коде:
    

```swift
Bundle.main.url(forResource: "FolderName/config", withExtension: "json")
```

#### 4️⃣ Проверка Build Phases → Copy Bundle Resources

- Target → Build Phases → **Copy Bundle Resources**
    
- Убедитесь, что нужный ресурс есть в списке.
    

---

### Пример исправления

**Было:**

```swift
let image = UIImage(named: "logo.png")
```

- `logo.png` отсутствует в Target → предупреждение `Resource logo.png not found`.
    

**Исправлено:**

1. Добавили `logo.png` в Target Membership.
    
2. Убедились, что он присутствует в **Copy Bundle Resources**.
    

```swift
let image = UIImage(named: "logo.png") // теперь без предупреждений
```

---
