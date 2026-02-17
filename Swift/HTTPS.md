**HTTPS** — это защищённая версия протокола **[[HTTP]]**, в которой весь трафик между клиентом (браузер, приложение) и сервером **шифруется** с помощью **TLS** (Transport Layer Security, ранее SSL).

### Основные отличия HTTP vs HTTPS

| Характеристика              | HTTP                              | HTTPS                                      |
|-----------------------------|-----------------------------------|--------------------------------------------|
| Шифрование                  | Нет                               | Да (TLS 1.2 или 1.3)                       |
| Порт по умолчанию           | 80                                | 443                                        |
| Защита от перехвата         | Нет (MITM-атаки)                  | Да (Man-in-the-Middle невозможен)          |
| Целостность данных          | Нет                               | Да (HMAC)                                  |
| Аутентификация сервера      | Нет                               | Да (сертификат)                            |
| Доверие браузера            | Предупреждение «Не защищено»      | Замок + «Безопасно»                        |
| SEO-ранжирование (Google)   | Пониженный приоритет              | Повышенный приоритет                       |
| Обязателен в 2026 году?     | Нет (но почти все браузеры ругают)| Да — де-факто стандарт                     |

### Как работает HTTPS (упрощённо)

1. Клиент (браузер/приложение) → серверу: «Привет, давай TLS»
2. Сервер → клиенту: «Вот мой сертификат (подписан CA)»
3. Клиент проверяет сертификат (через цепочку доверия → Let's Encrypt, Google Trust Services, DigiCert и т.д.)
4. Обмен ключами (Diffie-Hellman или ECDHE) → симметричный ключ
5. Всё последующее шифруется (AES-256-GCM обычно)

### Современные требования к HTTPS в 2026 году

- **TLS 1.3** — обязательный минимум (TLS 1.2 уже устарел)
- **HSTS** (HTTP Strict Transport Security) — заголовок `Strict-Transport-Security`
- **CAA** (Certification Authority Authorization) — защита от поддельных сертификатов
- **Certificate Transparency** — все сертификаты публично логируются
- **OCSP Stapling** или **OCSP Must-Staple** — проверка отзыва сертификата
- **ECDSA** или **Ed25519** вместо RSA (быстрее и безопаснее)
- **Let's Encrypt** / **ZeroSSL** — бесплатные сертификаты с автопродлением

### Преимущества HTTPS в 2026 году

1. **Конфиденциальность** — никто не увидит логин/пароль, токены, платежи
2. **Целостность** — данные не модифицируются в пути
3. **Аутентичность сервера** — защита от фишинга (пользователь видит замок)
4. **SEO** — Google ранжирует HTTPS-сайты выше
5. **Web Vitals & Core Web Vitals** — HTTPS влияет на скорость и безопасность
6. **PWA и Service Workers** — требуют HTTPS
7. **Браузеры ругают HTTP** — красный треугольник, «Не защищено», блокировка некоторых функций

### Как внедрить HTTPS в [[iOS]]-приложении (2026)

#### 1. App Transport Security (ATS)

В `Info.plist` почти всегда:

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
</dict>
```

Исключения (только если вынуждены):

```xml
<key>NSExceptionDomains</key>
<dict>
    <key>example.com</key>
    <dict>
        <key>NSExceptionAllowsInsecureHTTPLoads</key>
        <true/>
    </dict>
</dict>
```

**Лучшая практика**: вообще не использовать `NSAllowsArbitraryLoads` в продакшене.

#### 2. [[URLSession]] + HTTPS (самый частый код)

```swift
let url = URL(string: "https://api.example.com/users")!
var request = URLRequest(url: url)
request.httpMethod = "GET"

let (data, response) = try await URLSession.shared.data(for: request)

// Проверка HTTPS
if let httpResponse = response as? HTTPURLResponse,
   let url = httpResponse.url,
   url.scheme == "https" {
    print("Соединение защищено TLS")
}
```

#### 3. Проверка сертификата (Certificate Pinning)

```swift
class SecureSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Проверяем, совпадает ли публичный ключ с ожидаемым
        // (pinning публичного ключа или сертификата)
        let serverPublicKey = SecCertificateCopyPublicKey(certificate)
        // Сравниваем с известным ключом...
        
        completionHandler(.useCredential, URLCredential(trust: serverTrust))
    }
}
```

### Короткий итог (2026)

- **HTTPS — обязателен** для любого приложения, которое передаёт данные
- **TLS 1.3** — минимум
- **HSTS** и **Certificate Pinning** — must-have для безопасности
- **ATS** в iOS блокирует HTTP по умолчанию — это хорошо
- **[[Universal Link]]**, **App Clips**, **Associated Domains** — работают только по HTTPS

**Главное правило**:
> «Если твой API или сайт всё ещё на HTTP в 2026 году — ты нарушаешь безопасность пользователей и теряешь доверие.  
> HTTPS — не фича, а базовая гигиена.»
