**Домен** — это иерархическая текстовая метка, которая входит в состав **[[URL]]** и служит для идентификации ресурса в интернете (сайта, сервера, [[API]] и т.д.).

### Структура домена (слева направо)

| Часть                               | Пример в URL                         | Обязательна? | Описание                                                                                                                                                                         |
| ----------------------------------- | ------------------------------------ | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Протокол**                        | `https://`                           | Да           | Указывает протокол: [[HTTP]], [[HTTPS]], `ftp`, `mailto`, `myapp://` (custom scheme)                                                                                             |
| **Поддомен** (опционально)          | `api`, `www`, `blog`, `dev`          | Нет          | Уточняет подраздел или сервис. Может быть многоуровневым: `api.dev.company.com`                                                                                                  |
| **Основной (второй уровень) домен** | `example`, `google`, `apple`         | Да           | Имя компании, бренда или проекта                                                                                                                                                 |
| **TLD** (Top-Level Domain)          | `.com`, `.org`, `.ru`, `.app`, `.io` | Да           | Доменная зона верхнего уровня. Бывает:<br>• gTLD (общие): `.com`, `.net`, `.app`<br>• ccTLD (страновые): `.ru`, `.ua`, `.de`<br>• sTLD (спонсируемые): `.edu`, `.gov`, `.museum` |
| **Порт** (опционально)              | `:8080`                              | Нет          | Порт сервера (по умолчанию 80 для http, 443 для https)                                                                                                                           |

### Реальные примеры разборов

| URL                                      | Протокол | Поддомен(ы)       | Основной домен | TLD  | Порт |
|------------------------------------------|----------|-------------------|----------------|------|------|
| https://www.apple.com                    | https    | www               | apple          | .com | —    |
| https://api.developers.google.com/v1     | https    | api.developers    | google         | .com | —    |
| https://blog.company.ru/news             | https    | blog              | company        | .ru  | —    |
| myapp://profile/123?token=abc            | myapp    | —                 | —              | —    | —    |
| https://localhost:8080/debug             | https    | —                 | localhost      | —    | 8080 |

### Важные понятия 2026 года

- **gTLD** — общие домены верхнего уровня (.com, .app, .dev, .io, .xyz и сотни новых)
- **ccTLD** — страновые (.ru, .ua, .de, .jp)
- **new gTLD** — новые общие зоны (.tech, .shop, .online, .blog, .ai)
- **IDN** — интернационализированные домены (кириллица, арабский и т.д.) — `.рф`, `.укр`
- **Punycode** — преобразование IDN в ASCII (xn--80asehdb для .рф)

### [[deep link]] и Custom URL Schemes в [[iOS]]

В iOS домен часто используется в **deep linking**:

- Custom scheme: `myapp://profile/123`  
  → регистрируется в Info.plist → обрабатывается в `open url`

- Universal Links: `https://example.com/profile/123`  
  → требует apple-app-site-association на сервере  
  → обрабатывается в `continue userActivity`

Пример обработки:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    
    if url.scheme == "myapp" {
        handleCustomScheme(url)
    }
}

func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    
    handleUniversalLink(url)
}
```

### Короткий итог (2026)

- **Домен** = поддомен(ы) + основной домен + TLD
- **Полный URL** = протокол + домен + путь + query + fragment
- В iOS:
  - Custom scheme — для внутренних deep link
  - [[Universal Link]] — основной и рекомендуемый способ
- Новые gTLD и IDN активно используются
- Безопасность: всегда проверяй scheme/host в обработчиках

**Главное правило**:
> «В 2026 году Universal Links — основной способ deep linking в iOS.  
> Custom scheme — только fallback или внутренние переходы.  
> Всегда проверяй host и path, чтобы не открывать фишинговые ссылки.»
