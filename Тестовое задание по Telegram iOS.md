## 1. Определите архитектурные слои (UI / Domain / Data / Network).

Архитектура:
- UI - похож на MVC + Rx с собственными фасадами и слоями, с кастомным NavigationController + многомодульность
- Network - собственный Protocol Engine (контекст + сессия + транспорт) + RPC request service + Swift adapter с реактивным API
- Domain - монолитный модуль `TelegramCore` с корнем `Account`, фасадом `TelegramEngine` по фичам
- Data - кастомный embedded store Postbox: SQLCipher под абстракцией ValueBox, множество узких таблиц, строгая модель транзакций

Стек:
- Swift/ObjC, сборка Bazel, UI UIKit + AsyncDisplayKit + Display, данные Postbox/SQLCipher, сеть MTProto, асинхронность SwiftSignalKit, медиа FFmpeg/WebP/Lottie/rlottie, звонки WebRTC

## 2. Проследите один типичный поток данных (например: получение сообщения → локальный кеш → UI → push/notification)

сценарий для нового входящего сообщения по MTProto (когда приложение онлайн или поднимается фоном) и отдельно — ветка push, которая идёт параллельно и не обязана проходить через тот же порядок «сначала кеш, потом уведомление».

---

### 1. Сеть → разбор MTProto

|Этап|Файлы|
|---|---|
|Чтение TCP, расшифровка пакета|`submodules/MtProtoKit/Sources/MTTcpConnection.m` (`processReceivedData:...`)|
|Обработка входящего MTProto-сообщения|`submodules/MtProtoKit/Sources/MTProto.m` (`_processIncomingMessage:...`)|

---

### 2. `Api.Updates` → очередь апдейтов аккаунта

|Этап|Файлы|
|---|---|
|Реализация `MTMessageService`, из тела сообщения достаётся `Api.Updates`|`submodules/TelegramCore/Sources/State/UpdateMessageService.swift` (`mtProto:receivedMessage:...`, `addUpdates`)|
|Разбиение сырого списка `Api.Update` на группы|`submodules/TelegramCore/Sources/State/UpdateGroup.swift` (`groupUpdates`)|
|Регистрация сервиса на `MTProto`, подписка на `pipe`, `addUpdateGroups`|`submodules/TelegramCore/Sources/State/AccountStateManager.swift` (`reset`, `addUpdateGroups`, внутри `Impl`)|
|Точка входа аккаунта для апдейтов (в т.ч. из других подсистем)|`submodules/TelegramCore/Sources/Account/Account.swift` (например `addUpdates` / связка с `stateManager`)|

---

### 3. Локальный «кеш» = Postbox (SQLite + таблицы истории)

|Этап|Файлы|
|---|---|
|Применение собранного состояния к транзакции postbox (основная «запись в кеш»)|`submodules/TelegramCore/Sources/State/AccountStateManagementUtils.swift` (`replayFinalState`)|
|Транзакции, обновление сообщений, подписки на представления истории|`submodules/Postbox/Sources/Postbox.swift` (`updateMessage`, `aroundMessageHistoryViewForLocation`, …)|
|Хранение строк истории сообщений|`submodules/Postbox/Sources/MessageHistoryTable.swift`, рядом индексы/теги: `MessageHistoryIndexTable.swift`, `MessageHistoryTagsTable.swift`, …|
|Модель «среза» истории для UI|`submodules/Postbox/Sources/MessageHistoryView.swift`, `MessageHistoryViewState.swift`|

Дополнительно для частных случаев (редактирование/ответ API и т.п.): `submodules/TelegramCore/Sources/State/ApplyUpdateMessage.swift`.

---

### 4. UI: подписка на Postbox → экран чата

|Этап|Файлы|
|---|---|
|Обёртка над `postbox.aroundMessageHistoryViewForLocation` для аккаунта|`submodules/TelegramCore/Sources/State/AccountViewTracker.swift`|
|Сбор сигнала истории для конкретного чата|`submodules/TelegramUI/Sources/ChatHistoryViewForLocation.swift`|
|Узел списка сообщений, `MessageHistoryView` → ячейки|`submodules/TelegramUI/Sources/ChatHistoryListNode.swift`|

Контроллер чата обычно рядом в том же модуле (`ChatController` и связанные файлы в `submodules/TelegramUI/Sources/`), но ключевой поток данных в список — через цепочку выше.

---

### 5. Push / уведомления (отдельная ветка от сервера через APNs)

Здесь нет строгой цепочки «сообщение → postbox → push»: push приходит от Apple Push по решению сервера; postbox может использоваться позже (например, для расшифровки/обогащения в extension).

|Этап|Файлы|
|---|---|
|Service Extension (модификация контента уведомления)|`Telegram/NotificationService/Sources/NotificationService.swift`|
|Обработка remote notification в приложении|`submodules/TelegramUI/Sources/AppDelegate.swift` (`application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`)|
|Общая логика уведомлений/аккаунтов на стороне UI|`submodules/TelegramUI/Sources/SharedNotificationManager.swift`|
|Строки шаблонов текста пушей (локализация)|`Telegram/Telegram-iOS/en.lproj/Localizable.strings` (ключи `PUSH_*`)|

---

## Что изучали

Сначала искали **одну** осмысленную тему среди UX, производительности, синхронизации, offline, уведомлений и навигации/deep link — не чеклист на весь репозиторий, а точку с явным кодом и последствиями.

По ходу этого просматривали:

- **Навигация и ссылки:** `AppDelegate.swift` (биндинги `openUrl` / `openUniversalUrl` / `canOpenUrl`), `OpenUrl.swift` (в т.ч. `handleInternetUrl`, вызов `openUniversalUrl` с fallback на `continueHandling`), упоминания `UrlHandling`, `BrowserUI` (`BrowserWebContent` — политика навигации по `tg://` / t.me), фрагменты `WebUI` (`WebAppController`).
- **Расширения:** `ShareExtensionContext`, `NotificationContentContext` — урезанные `TelegramApplicationBindings` (в т.ч. заглушки `openUniversalUrl`), чтобы понять, не «ломается» ли сценарий только в extension.
- **Ядро / сеть (поверхностно):** `TelegramCore` — куски `Network`, `PendingMessageManager`, обработка ошибок перевода/саммари и т.п. как фоновые кандидаты, без углубления.
- **Поиск маркеров:** по репозиторию по `FIXME`/`TODO` в `.swift` — **почти пусто**, явных «известных багов» в комментариях не оперлись.
- **Уведомления → чат:** `ApplicationContext` (открытие чата при активном приложении), затем **`AppDelegate.userNotificationCenter(_:didReceive:...)`** — где и нашли разрыв в логике.

## Какие слои / модули затронули

| Слой / область | Модуль / путь (в основном) |
|----------------|----------------------------|
| UI приложения, lifecycle, push | `submodules/TelegramUI/Sources/` — `AppDelegate.swift`, `OpenUrl.swift`, `ApplicationContext.swift` |
| Контекст аккаунта / биндинги приложения | `submodules/AccountContext/` (сигнатуры `TelegramApplicationBindings`) |
| In-app browser / WebView | `submodules/BrowserUI/`, `submodules/WebUI/` |
| Парсинг / обработка URL (ядро) | `submodules/UrlHandling/` (поиск), `TelegramCore` (точечно) |
| Расширения | `TelegramUI/Components/ShareExtensionContext/`, `TelegramUI/Sources/NotificationContentContext.swift` |

Глубоко разобран один узкий участок: **обработка тапа по push в основном приложении** в `AppDelegate`.

## Почему выбрали именно эту проблему

1. **Одна чёткая тема** — вы просили не «всё подряд», а один артефакт; здесь один условный блок и понятное следствие.
2. **Реальная логическая дыра в коде:** при наличии `userInfo["url"]` ветка с `openChatWhenReady` **вообще не выполняется**; если `URL(string:)` вернул `nil`, **не срабатывает ничего** — это не догадка, а прямое следствие структуры `if / else`.
3. **Сильный UX при срабатывании:** пользователь ожидает переход в чат или по ссылке, а получает «пустое» открытие приложения.
4. **Малый объём исправления:** локальное изменение в одном обработчике, без затягивания всего стека deep link / `OpenUrl`.
5. **Альтернативы были размытее:** расхождение `openUrl` vs `openUniversalUrl` для схемы `tg`, case-sensitivity схемы, заглушки в extension — либо спорно как баг, либо осознанный дизайн sandbox; маркеры `FIXME` не дали опоры.

Итого: выбор — **компромисс между доказуемостью по коду, влиянием на UX (уведомления + навигация) и узким scope** для одного issue.