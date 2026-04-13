**NSMicrophoneUsageDescription** — это **обязательный ключ** в файле **Info.plist** вашего iOS-приложения, который объясняет пользователю, **зачем** приложению нужен доступ к микрофону.

Без этого ключа (или если значение пустое/неинформативное) приложение **крашнется** при первой попытке использовать микрофон — начиная с **[[iOS]] 10** (2016) и по сегодняшний день (2026).

### Почему ключ обязателен

Apple требует явного объяснения для всех privacy-sensitive функций.  
Микрофон относится к чувствительным данным, поэтому без корректного описания:

- При вызове `AVAudioSession.requestRecordPermission`, [[AVCaptureSession]] (видео со звуком), [[UIImagePickerController]] с видео, `SFSpeechRecognizer` и т.д. приложение **упадёт** с типичной ошибкой:

```
This app has crashed because it attempted to access privacy-sensitive data without a usage description.  
The app's Info.plist must contain an NSMicrophoneUsageDescription key with a string value explaining to the user how the app uses this data.
```

### Как правильно добавить NSMicrophoneUsageDescription

#### 1. Через Xcode (самый удобный и рекомендуемый способ)

1. Откройте проект → выберите target → вкладка **Info**
2. Нажмите **+** в разделе **Custom iOS Target Properties**
3. В поиске введите «Microphone» или найдите **Privacy - Microphone Usage Description**
4. В поле **Value** напишите понятное объяснение на языке пользователя (на английском и/или русском)

Примеры хороших текстов (2026 стандарт — проходят App Review без проблем):

```text
"Приложению нужен доступ к микрофону, чтобы вы могли записать голосовое сообщение или видео."
```

```text
"Микрофон используется для записи аудио в заметках и голосовых сообщениях."
```

```text
"Доступ к микрофону требуется для видеозвонков и записи звука в приложении."
```

Плохие примеры (App Review отклонит или пользователь откажет в доступе):

```text
"Для работы приложения"
"Microphone access"
"Нужен микрофон"
```

#### 2. Через Info.plist напрямую (XML-вид)

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Приложению нужен доступ к микрофону, чтобы вы могли записать голосовое сообщение или видео.</string>
```

#### 3. Локализация (если приложение мультиязычное)

Создайте файл **InfoPlist.strings** в папке локализации (например ru.lproj):

```text
/* ru.lproj/InfoPlist.strings */
"NSMicrophoneUsageDescription" = "Приложению нужен доступ к микрофону, чтобы вы могли записать голосовое сообщение или видео.";
```

А в **Base.lproj/InfoPlist.strings** (английский):

```text
"NSMicrophoneUsageDescription" = "The app needs microphone access to record voice messages or video.";
```

### Когда обязательно добавлять NSMicrophoneUsageDescription

| Функция / API                              | Требуется ключ? | Дополнительные ключи |
|--------------------------------------------|-----------------|-----------------------|
| `UIImagePickerController` (sourceType = .camera + видео) | Да              | + NSCameraUsageDescription |
| `AVCaptureSession` с аудио-треком          | Да              | + NSCameraUsageDescription (если видео) |
| `AVAudioRecorder`                          | Да              | — |
| `SFSpeechRecognizer` (распознавание речи)  | Да              | — |
| `AVSpeechSynthesizer`                      | Нет             | — (только синтез, не запись) |
| `ReplayKit` (запись экрана со звуком)      | Да              | — |
| `CallKit` / VoIP                           | Да              | — |

### Лучшие практики NSMicrophoneUsageDescription в 2026 году

- **Текст должен быть конкретным** — пользователь должен сразу понять, зачем именно микрофон нужен  
- **Не используйте** общие фразы типа "Для записи звука" — App Review требует понятного объяснения  
- **Добавляйте** ключ **даже если** пока не используете микрофон — если планируете в будущем  
- **Для SwiftUI** — ключ всё равно нужен в Info.plist ([[SwiftUI]] не меняет требования приватности)  
- **Тестируйте** на устройстве без разрешения — приложение должно **крашнуться** только если ключ отсутствует  
- **Для App Review** — всегда добавляйте ключ, даже если микрофон используется косвенно (например, через сторонние SDK)  
- **Privacy Manifest** — **не требуется** для NSMicrophoneUsageDescription (это описание, а не сбор данных)  
- **Документируйте** в Info.plist:

```xml
<!-- NSMicrophoneUsageDescription -->
<!-- Объяснение для пользователя: зачем приложению нужен микрофон -->
<key>NSMicrophoneUsageDescription</key>
<string>Приложению нужен доступ к микрофону, чтобы вы могли записать голосовое сообщение или видео.</string>
```

**Короткий итог 2026**:
> `NSMicrophoneUsageDescription` — **обязательный** ключ в Info.plist, который объясняет, зачем приложению нужен микрофон.  
> В 2026 году:  
> - без него приложение **крашнется** при попытке использовать микрофон  
> - текст должен быть **конкретным** и **понятным** пользователю  
> - добавляется в [[Xcode]] → Info → Privacy - Microphone Usage Description  
> - если используете камеру + микрофон → добавьте ещё и NSCameraUsageDescription  
> Это **одна из самых частых** причин крашей и отказов в App Review.
