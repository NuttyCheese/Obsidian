#system_design
## Определение

**MITM (Man-in-the-Middle) атака** — это тип кибератаки, при которой злоумышленник **перехватывает, изменяет или подслушивает данные**, передаваемые между клиентом ([[iOS]]-приложением) и сервером.

**Цель защиты:** гарантировать, что все сетевые запросы проходят **только к доверенным серверам** и данные не подменяются.

---

## 1. Основные методы защиты

| Метод                                     | Описание                                                     | Применение в iOS                                            |
| ----------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| **[[HTTPS]] (TLS/SSL)**                   | Шифрование трафика с использованием сертификатов             | [[URLSession]] автоматически поддерживает TLS               |
| **Certificate Pinning**                   | Привязка приложения к конкретному сертификату сервера        | Проверка сертификата в коде до установления соединения      |
| **Public Key Pinning**                    | Привязка к публичному ключу сервера вместо всего сертификата | Более гибко, при смене сертификата сохраняется безопасность |
| **HSTS (HTTP Strict Transport Security)** | Принудительное использование HTTPS                           | Настраивается на сервере                                    |
| **OAuth/JWT + TLS**                       | Авторизация и шифрование данных                              | Используется для безопасного [[API]]-доступа                |

---

## 2. Certificate Pinning в iOS

### Принцип работы

- Приложение хранит **сертификат или публичный ключ сервера**.
    
- При установлении HTTPS соединения проверяется, совпадает ли сертификат сервера с локальным.
    
- Если сертификат не совпадает → соединение разрывается, предотвращая MITM.
    

### Пример с [[URLSessionDelegate]]

```swift
func urlSession(_ session: URLSession, 
                didReceive challenge: URLAuthenticationChallenge, 
                completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {

    guard let serverTrust = challenge.protectionSpace.serverTrust,
          let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
        completionHandler(.cancelAuthenticationChallenge, nil)
        return
    }

    let remoteCertificateData = SecCertificateCopyData(certificate) as Data
    let localCertificateData = NSData(contentsOfFile: Bundle.main.path(forResource: "server", ofType: "cer")!)! as Data

    if remoteCertificateData == localCertificateData {
        let credential = URLCredential(trust: serverTrust)
        completionHandler(.useCredential, credential)
    } else {
        completionHandler(.cancelAuthenticationChallenge, nil)
    }
}
```

**Объяснение:**

- `serverTrust` → сертификат, который прислал сервер.
    
- Сравниваем его с локально сохранённым сертификатом (`server.cer`).
    
- Если совпадает → доверяем соединению, иначе — отклоняем.
    

---

## 3. Public Key Pinning

- Сохраняется только **публичный ключ сертификата**, а не весь сертификат.
    
- Позволяет обновлять сертификаты на сервере без потери доверия.
    
- Процесс аналогичен Certificate Pinning, но сравнивается только ключ.
    

---

## 4. Дополнительные меры защиты

1. **TLS 1.2+** → использовать только современные протоколы шифрования.
    
2. **Отказ от небезопасных библиотек** → избегать устаревших SSL-библиотек.
    
3. **Валидация hostname** → проверка соответствия имени сервера.
    
4. **Обработка ошибок сетевого соединения** → при любых подозрительных событиях разрывать соединение.
    
5. **Мониторинг и логирование** → отслеживать подозрительные попытки соединения.
    

---

## 5. Best Practices для iOS

- **Всегда использовать HTTPS** → даже для внутренних API.
    
- **Certificate / Public Key Pinning** → защита от MITM.
    
- **Secure storage токенов** → хранить access и refresh token в Keychain.
    
- **Регулярная проверка сертификатов** → обновление в приложении при изменении серверного сертификата.
    
- **Соблюдать принцип наименьших прав** → минимизировать чувствительные данные в запросах.
    

---

## 6. Итог

- **MITM-атаки** представляют серьёзную угрозу сетевому взаимодействию.
    
- **Основные методы защиты:** TLS, Certificate/Public Key Pinning, HSTS, Secure токены.
    
- **iOS** предоставляет инструменты ([[URLSession]], `SecTrust`, [[Keychain]]) для реализации защиты.
    
- Правильная комбинация этих методов позволяет **обеспечить безопасное взаимодействие с сервером и защиту данных пользователей**.
    

---
