#automation #third-party_library 
## 📘 Определение

**SwiftLint** — это инструмент для **статического анализа кода [[Swift]]**, который проверяет проект на соответствие выбранным **стилевым правилам** (linting).  
Помогает поддерживать **чистый, единообразный код**, обнаруживать потенциальные ошибки и предупреждать о нарушении стиля.  
Относится к **Development / Code Quality / Swift**.

---

## 🔹 Примеры кода и использования

### 1. Установка SwiftLint через Homebrew

```bash
brew install swiftlint
```

---

### 2. Проверка проекта вручную

```bash
cd MySwiftProject
swiftlint
```

---

### 3. Настройка правил через `.swiftlint.yml`

```yaml
disabled_rules: # отключаем правила
  - trailing_whitespace
  - force_cast

opt_in_rules: # включаем экспериментальные
  - empty_count
  - explicit_init

line_length: 120 # максимальная длина строки
excluded: # исключаем папки
  - Pods
  - Carthage
```

---

### 4. Интеграция SwiftLint в [[Xcode]] через Run Script

```bash
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "SwiftLint не установлен, пропускаем..."
fi
```

---

### 5. Примеры предупреждений SwiftLint

```swift
let name: String? = nil
print(name!) // warning: force unwrapping should be avoided
```

```swift
var  a = 5 // warning: variable name should be lowerCamelCase
```

SwiftLint поможет поддерживать **чистый, безопасный и читаемый код**.
