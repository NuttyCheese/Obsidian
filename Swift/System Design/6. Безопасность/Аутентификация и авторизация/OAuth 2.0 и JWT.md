#system_design
## Определение

**Аутентификация и авторизация** — процессы проверки личности пользователя и предоставления доступа к ресурсам приложения.

- **OAuth 2.0** → протокол авторизации, позволяющий приложению получать доступ к ресурсам пользователя без хранения его пароля.
    
- **JWT (JSON Web Token)** → компактный токен, который используется для передачи информации о пользователе и правах доступа между клиентом и сервером.
    

> В [[iOS]] часто используются совместно: OAuth 2.0 выдаёт токены (в том числе JWT), которые приложение использует для запросов к серверу.

---

## 1. OAuth 2.0

### Основные роли

|Роль|Описание|
|---|---|
|**Resource Owner**|Пользователь, владеющий ресурсами (например, аккаунт Google)|
|**Client**|Приложение (iOS app), запрашивающее доступ к ресурсам|
|**Authorization Server**|Сервер, выдающий токены после аутентификации пользователя|
|**Resource Server**|Сервер, предоставляющий доступ к защищённым данным через токен|

### Основные потоки (flows)

1. **Authorization Code Flow (с PKCE)** — безопасный поток для мобильных приложений:
    
    - Пользователь авторизуется через браузер или веб-вью.
        
    - Приложение получает **authorization code**.
        
    - Обменивает код на **access token** и **refresh token**.
        
2. **Implicit Flow** — устаревший, менее безопасный, не рекомендуется для iOS.
    
3. **Client Credentials Flow** — для серверных приложений без пользователя.
    
4. **Refresh Token** — позволяет обновлять **access token** без повторной авторизации пользователя.
    

### Пример iOS с Authorization Code Flow + PKCE

```swift
import AuthenticationServices

let authURL = URL(string: "https://authserver.com/authorize?client_id=CLIENT_ID&response_type=code&scope=profile")!
let session = ASWebAuthenticationSession(url: authURL, callbackURLScheme: "myapp") { callbackURL, error in
    guard let callbackURL = callbackURL else { return }
    
    // Получение authorization code
    let code = URLComponents(string: callbackURL.absoluteString)?
        .queryItems?.first(where: { $0.name == "code" })?.value
    
    // Обмен на access token
    exchangeCodeForToken(code: code)
}
session.presentationContextProvider = self
session.start()
```

---

## 2. JWT (JSON Web Token)

### Структура

JWT состоит из **трёх частей, разделённых точками**:

```
HEADER.PAYLOAD.SIGNATURE
```

1. **Header** → тип токена и алгоритм подписи (`{"alg": "HS256","typ": "JWT"}`).
    
2. **Payload** → данные (claims) о пользователе и правах доступа:
    
    - `sub` → идентификатор пользователя
        
    - `exp` → время истечения токена
        
    - `iat` → время выдачи
        
    - `scope` → права доступа
        
3. **Signature** → цифровая подпись для проверки целостности токена
    

### Пример декодирования JWT на [[Swift]]

```swift
import Foundation

func decode(jwt: String) -> [String: Any]? {
    let segments = jwt.split(separator: ".")
    guard segments.count == 3 else { return nil }
    
    let payloadSegment = segments[1]
    var payload = payloadSegment.replacingOccurrences(of: "-", with: "+")
    payload = payload.replacingOccurrences(of: "_", with: "/")
    
    let paddingLength = 4 - payload.count % 4
    payload += String(repeating: "=", count: paddingLength)
    
    guard let data = Data(base64Encoded: payload) else { return nil }
    return try? JSONSerialization.jsonObject(with: data) as? [String: Any]
}
```

---

## 3. Использование OAuth 2.0 + JWT в iOS

- **Access token** → используется для запросов к серверу в `Authorization: Bearer <token>`.
    
- **Refresh token** → обновление access token при истечении срока действия.
    
- **JWT** → компактный и самодостаточный токен, проверяемый на сервере без отдельного запроса.
    

### Пример запроса с токеном

```swift
var request = URLRequest(url: URL(string: "https://api.server.com/user")!)
request.setValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")

URLSession.shared.dataTask(with: request) { data, response, error in
    // Обработка ответа
}.resume()
```

---

## 4. Best Practices

1. **Использовать Authorization Code Flow + PKCE** для мобильных приложений.
    
2. **Не хранить пароли** → использовать токены OAuth.
    
3. **Secure storage** → хранить токены в **Keychain**.
    
4. **Обновление токена через refresh token** → избегать повторной авторизации.
    
5. **Проверка срока действия токена** → `exp` в JWT.
    
6. **Минимизация scope** → давать приложению только необходимые права.
    
7. **Защита передачи токенов** → только через [[HTTPS]].
    

---

## Итог

- **OAuth 2.0** → безопасная авторизация без хранения пароля.
    
- **JWT** → компактный токен для передачи информации о пользователе и правах доступа.
    
- Использование вместе в iOS: **PKCE для мобильных клиентов, access token для [[API]], refresh token для обновления**.
    
- Best practices: Keychain, HTTPS, минимальные права, контроль срока действия.
    

---
