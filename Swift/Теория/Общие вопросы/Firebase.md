#general_theory #third-party_library
**Firebase** — это **Backend-as-a-Service** (BaaS) платформа от Google, которая помогает быстро создавать, масштабировать и монетизировать мобильные и веб-приложения.

### Ключевые сервисы Firebase (актуально на 2026)

| Сервис                             | Назначение                                                                      | Основные возможности (2026)                                  | Альтернативы               |
| ---------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| **Realtime Database**              | [[Типы баз данных#2. **NoSQL (нереляционные)**\|NoSQL]] база в реальном времени | [[JSON]]-дерево, оффлайн-синхронизация, правила безопасности | Firestore                  |
| **Cloud Firestore**                | Масштабируемая NoSQL база документов                                            | Коллекции, подколлекции, индексы, оффлайн, слушатели         | Realtime DB                |
| **Firebase Authentication**        | Аутентификация пользователей                                                    | Email, Google, Apple, Phone, Anonymous, OAuth, MFA           | Supabase Auth, Auth0       |
| **Cloud Storage**                  | Хранение файлов (изображения, видео, бинарники)                                 | Загрузка/скачивание, resumable uploads, правила              | AWS S3, Supabase Storage   |
| **Firebase Cloud Messaging (FCM)** | Push-уведомления и сообщения                                                    | Топики, группы устройств, data + notification payload        | OneSignal, APNs            |
| **Firebase Analytics**             | Сбор и анализ поведения пользователей                                           | События, аудитория, конверсии, BigQuery интеграция           | Amplitude, Mixpanel        |
| **Crashlytics**                    | Отслеживание крашей и не-фатальных ошибок                                       | Стэктрейсы, Breadcrumbs, AI-анализ причин                    | Sentry, Bugsnag            |
| **Cloud Functions** (2nd gen)      | Serverless функции на Node.js, Python, Go и др.                                 | Event-driven, [[HTTP]], Pub/Sub, Firestore триггеры          | AWS Lambda, Supabase Edge  |
| **Remote Config**                  | Динамическая конфигурация приложения без релиза                                 | A/B-тестирование, персонализация, feature flags              | LaunchDarkly               |
| **A/B Testing**                    | Эксперименты над пользователями                                                 | Интеграция с Remote Config и Analytics                       | Optimizely                 |
| **Performance Monitoring**         | Отслеживание производительности (задержки сети, загрузка экранов)               | Трейсинг, метрики, автоматические экраны                     | Sentry, New Relic          |
| **App Distribution**               | Распространение бета-версий приложения                                          | Через TestFlight-подобный интерфейс                          | [[TestFlight]], App Center |
| **Firebase ML**                    | Машинное обучение на устройстве и в облаке                                      | AutoML, кастомные модели, Vision API, NLP                    | Core ML, TensorFlow Lite   |
| **Dynamic Links**                  | Глубокие ссылки (deep linking)                                                  | Отложенный deep link, кросс-платформенный                    | Branch, AppsFlyer          |
| **In-App Messaging**               | Сообщения внутри приложения                                                     | Карточки, баннеры, модальные окна                            | Braze, Airship             |

### Самые популярные связки Firebase в 2026

| Тип приложения                     | Основные сервисы Firebase                              | Почему именно эти |
|------------------------------------|--------------------------------------------------------|-------------------|
| Социальная сеть / чат              | Firestore + Auth + Storage + FCM + Cloud Functions     | Реал-тайм, push, файлы, серверная логика |
| E-commerce / маркетплейс           | Firestore + Auth + Analytics + Remote Config + A/B     | Каталог, пользователи, аналитика, персонализация |
| Игры (казуальные)                  | Realtime DB + Auth + Analytics + Crashlytics + Remote Config | Быстрый реал-тайм, краши, A/B |
| Корпоративные приложения           | Firestore + Auth (Google SSO) + Cloud Functions + Security Rules | Безопасность, интеграция с G Suite |
| MVPs и стартапы                    | Firestore + Auth + Hosting + Functions + Analytics     | Быстрый запуск, минимум инфраструктуры |

### Плюсы Firebase в 2026 году

- Почти нулевая инфраструктура (serverless)
- Отличная интеграция с Google Cloud (BigQuery, Functions 2nd gen, Vertex AI)
- Реал-тайм из коробки (Firestore / Realtime DB)
- Бесплатный уровень щедрый для старта
- Хорошая документация и SDK для SwiftUI, [[UIKit]], Flutter, React Native

### Минусы и ограничения (реальность 2026)

- **Vendor lock-in** — миграция на другой бэкенд сложная
- **Цены** растут быстро при масштабе (особенно Firestore reads/writes)
- **Холодный старт** Cloud Functions 2nd gen всё ещё заметен (хотя лучше, чем 1st gen)
- **Ограничения** по количеству одновременных подключений и запросов
- **Не все операции** можно делать эффективно (нет сложных join’ов в Firestore)
- **Зависимость от Google** — риски изменения политики/цен

### Современные альтернативы Firebase (2026)

| Альтернатива                        | Сильные стороны                          | Слабые стороны              | Когда выбрать вместо Firebase |
| ----------------------------------- | ---------------------------------------- | --------------------------- | ----------------------------- |
| **Supabase**                        | Open-source, PostgreSQL, реал-тайм, auth | Меньше сервисов, моложе     | Хочешь SQL и open-source      |
| **Appwrite**                        | Полностью self-hosted, open-source       | Нужно поддерживать сервер   | Полный контроль и приватность |
| **Back4App**                        | Parse Server + Live Query + GraphQL      | Меньше экосистемы           | Parse-стиль разработки        |
| **AWS Amplify**                     | Глубокая интеграция с AWS                | Сложнее, vendor lock-in AWS | Уже в AWS-экосистеме          |
| **Vercel + Supabase / PlanetScale** | Frontend + DB + Functions                | Нужно собирать вручную      | Next.js / React приложения    |

**Короткое правило выбора в 2026**:
> «Если нужен быстрый запуск, реал-тайм и интеграция с Google — бери **Firebase**.  
> Если хочешь SQL, open-source или полный контроль — смотри на **Supabase** / **Appwrite**.  
> Если уже в AWS — **Amplify**.  
> Если чистый frontend — **Vercel + любая DB**.»
