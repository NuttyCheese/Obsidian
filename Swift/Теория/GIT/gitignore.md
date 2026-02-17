**.gitignore** — это обычный текстовый файл (без расширения), который лежит в корне репозитория и говорит [[Git]]:

> «Эти файлы и папки никогда не добавляй в индекс и не коммить их».

Git **не отслеживает** их, они не попадают в историю, не показываются в `git status`, не мешают `git add .`.

### 2. Почему .gitignore критически важен в [[iOS]]/[[Swift]]

| Проблема без .gitignore                  | Последствия в iOS-проекте                                |
| ---------------------------------------- | -------------------------------------------------------- |
| Коммит DerivedData, xcuserdata           | Репозиторий разрастается до гигабайт, конфликты настроек |
| Коммит Pods/, [[Carthage]]/Checkouts     | Раздувание репозитория до 1–5 ГБ                         |
| Коммит .xcframework, .swiftpm, build/    | Дублирование зависимостей, конфликты                     |
| Коммит API-ключей, .env, личных настроек | Утечка секретов → взлом аккаунтов                        |
| Коммит .DS_Store, Thumbs.db              | Мусор в истории, конфликты на macOS/Windows              |

### 3. Структура и синтаксис .gitignore (2026)

| Правило / Синтаксис               | Что делает                                      | Пример в .gitignore                     |
|-----------------------------------|--------------------------------------------------|------------------------------------------|
| `*.log`                           | Игнорировать все файлы с расширением .log        | *.log                                    |
| `build/`                          | Игнорировать папку build и всё внутри            | build/                                   |
| `!important.txt`                  | Исключение из игнора (force include)             | !important.txt                           |
| `**/DerivedData/`                 | Игнорировать DerivedData в любой вложенности     | **/DerivedData/                          |
| `xcuserdata/`                     | Игнорировать пользовательские настройки Xcode    | xcuserdata/                              |
| `*.pbxproj`                       | Игнорировать проектные файлы Xcode               | *.pbxproj                                |
| `# комментарий`                   | Комментарий                                      | # Xcode generated files                  |

### 4. Готовый .gitignore для Swift/iOS-проекта 2026

```gitignore
# Xcode
*.xcuserstate
*.xcuserdata/
*.moved-aside
*.xcscmblueprint
*.xccheckout
xcuserdata/
DerivedData/
build/
*.moved-aside
*.xcuserstate
*.xcscmblueprint
*.xccheckout

# Swift Package Manager
.swiftpm/
.build/
Packages.resolved

# CocoaPods
Pods/
Podfile.lock

# Carthage
Carthage/Checkouts/
Carthage/Build/

# Fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/
fastlane/test_output/

# Logs и временные файлы
*.log
*.swp
*~
.DS_Store
.AppleDouble
.LSOverride
Thumbs.db
ehthumbs.db
Icon?

# Ключи, секреты, .env
.env
*.key
*.p12
*.mobileprovision
*.cer
secrets.xcconfig

# Тесты и coverage
*.gcda
*.gcno
*.profdata
coverage/

# Документация и артефакты
docs/
*.dSYM/
*.ipa
*.app
*.xcarchive/

# macOS системные файлы
.DS_Store
.AppleDouble
.LSOverride
Icon?
._*

# Исключения (если нужно коммитить что-то из вышеперечисленного)
!Podfile
!Podfile.lock  # если вы хотите коммитить lock-файл
!fastlane/Fastfile
!fastlane/Appfile
```

### 5. Глобальный .gitignore (рекомендуется для всех проектов)

```bash
# Создаём глобальный .gitignore
git config --global core.excludesfile ~/.gitignore_global

# Содержимое ~/.gitignore_global
.DS_Store
Thumbs.db
*.swp
*~
.AppleDouble
.LSOverride
Icon?
ehthumbs.db
```

### 6. Реальные сценарии и шаблоны 2026 года

#### Сценарий 1 — Чистый Swift Package Manager проект

```gitignore
# SPM
.build/
.swiftpm/
Packages.resolved

# Xcode (если есть)
*.xcworkspace/
*.xcodeproj/
xcuserdata/
DerivedData/
```

#### Сценарий 2 — Проект с [[CocoaPods]] + [[SPM]]

```gitignore
Pods/
Podfile.lock  # можно коммитить, но часто игнорируют
.swiftpm/
.build/
DerivedData/
xcuserdata/
```

#### Сценарий 3 — [[SwiftUI]] + [[Firebase]] + [[Fastlane]]

```gitignore
# Firebase
GoogleService-Info.plist
firebase.json

# Fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/
fastlane/test_output/

# Derived и Xcode
DerivedData/
xcuserdata/
*.moved-aside
```

### 7. Таблица: что чаще всего игнорировать в iOS/Swift

| Категория              | Что игнорировать                                 | Почему                            |
| ---------------------- | ------------------------------------------------ | --------------------------------- |
| [[Xcode]] генерируемое | xcuserdata/, DerivedData/, *.xcscmblueprint      | Личные настройки, кэш сборки      |
| Зависимости            | Pods/, Carthage/Checkouts/, .build/, .swiftpm/   | Бинарники, не нужны в git         |
| Секреты и ключи        | .env, *.p12, *.mobileprovision, secrets.xcconfig | Утечка → взлом                    |
| Логи и временные файлы | *.log, *~, .DS_Store, Thumbs.db                  | Мусор, конфликты macOS/Windows    |
| Артефакты сборки       | *.ipa, *.app, *.xcarchive, *.dSYM                | Генерируются каждый раз           |
| Fastlane / CI          | fastlane/report.xml, screenshots/, test_output/  | Результаты CI, не нужны в истории |

### 8. Лучшие практики .gitignore 2026

- Используйте **официальный шаблон** от [GitHub](https://github.com/github/gitignore/blob/main/Swift.gitignore)
- Добавьте **глобальный .gitignore** для системных файлов (.DS_Store и т.д.)
- **НЕ** коммитьте Podfile.lock, если команда большая (конфликты) — или коммитьте, если нужна точная воспроизводимость
- Для **Swift Package Manager** — почти всегда коммитьте `Package.resolved`
- Для **секретов** — используйте .gitignore + .env + git-crypt / git-secret
- Проверяйте `git status --ignored` — чтобы увидеть, что попало в игнор
- Используйте **lefthook** / **pre-commit** — чтобы проверять .gitignore перед коммитом

**Короткий девиз 2026**:
> «.gitignore — это фильтр чистоты репозитория.  
> Игнорируй всё, что генерируется, содержит секреты или занимает >10 МБ — и твой git clone будет весить 5–50 МБ вместо 2 ГБ.»
