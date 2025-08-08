### Интеграция iHealth SDK (Android и iOS)

Эта инструкция описывает полный процесс подготовки, подключения и работы с устройствами iHealth (на примере тонометра iHealth BP5S) в мобильных приложениях на Android и iOS. Включены требования к лицензии, загрузка SDK, настройка проекта и базовые сценарии работы с прибором.

- **Где регистрироваться**: `https://dev.ihealthlabs.com/account/sign-up`
- **Документация (Android, BP5S)**: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/android/quickstart/`, `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/android/blood_pressure/bp5s`
- **Документация (iOS, BP5S)**: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/ios/quickstart`, `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/ios/blood_pressure/bp5s`
- **Скачать актуальные SDK**: `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/android`, `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/ios`
- **Альтернатива**: можно взять SDK из этого репозитория в папке `sdk` (`sdk/android`, `sdk/ios`).

См. детальные платформенные инструкции:
- Android: см. `readme-android.md`
- iOS: см. `readme-ios.md`

---

### 1) Регистрация и выпуск лицензии

1. Зарегистрируйтесь на портале разработчика iHealth: `https://dev.ihealthlabs.com/account/sign-up`.
2. Войдите, откройте раздел Enterprise Development → App Management → Add New App.
3. Укажите:
   - **Application Name**: название вашего приложения.
   - **Package Name (Android)**: точный `applicationId` из вашего `build.gradle`.
   - **Bundle Identifier (iOS)**: точный идентификатор бандла из проекта Xcode.
4. Дождитесь активации. Проверка лицензии выполняется по точному совпадению идентификаторов. При расхождении SDK не пройдет валидацию.
5. Скачайте файл лицензии (обычно `.pem`) в разделе вашего приложения.

Примечание: активация может занимать 1–3 рабочих дня. При задержках — свяжитесь с вашим контактным лицом iHealth.

---

### 2) Загрузка SDK

Варианты:
- Скачайте свежую версию SDK с портала:
  - Android: `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/android`
  - iOS: `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/ios`
- Или используйте копии из текущего репозитория: папка `sdk/android`, `sdk/ios`.

Содержимое SDK включает библиотеки, зависимости и примеры использования. Детали интеграции приведены в платформа-специфичных файлах.

---

### 3) Общий процесс интеграции

На высоком уровне процесс одинаков для Android и iOS:
1. Добавьте SDK-файлы в проект (библиотеки, заголовки/фреймворки).
2. Поместите **лицензионный файл** в проект (Android: `app/src/main/assets`, iOS: в основной бандл).
3. Инициализируйте SDK при старте приложения.
4. Выполните **валидацию лицензии** до взаимодействия с устройствами.
5. Настройте права/разрешения Bluetooth и фоновые режимы (iOS), запросите runtime‑разрешения (Android).
6. Выполните **сканирование** BLE-устройств, **подключение** к BP5S и дальнейшее управление измерениями.

См. платформенные детали и примеры кода в `readme-android.md` и `readme-ios.md`.

---

### 4) Работа с устройствами iHealth (на примере BP5S)

Ключевые этапы:
- **Сканирование** устройств определенного типа (BP5S).
- **Подключение** по найденному MAC/ID.
- Получение **сервиса управления** устройством (контроллера), чтение батареи/функций.
- Запуск **измерения** и обработка событий/нотификаций SDK с результатами.
- Завершение работы: остановка измерения, отключение, снятие подписок/обработчиков.

Поскольку точные события и JSON‑ключи завязаны на профиль устройства, используйте актуальные константы из официальной документации для вашей версии SDK (см. ссылки выше). Минимальные примеры приведены в platform‑гайдах.

---

### 5) Безопасность и соответствие требованиям платформ

- Android 12+ требует явных Bluetooth‑разрешений (`BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT` и др.). Для Android 10/11 используются разрешения геолокации для BLE‑сканирования. См. подробности в `readme-android.md`.
- На iOS добавьте ключи конфиденциальности Bluetooth в `Info.plist` и при необходимости фоновые режимы (`bluetooth-central`). См. `readme-ios.md`.

---

### 6) Поддержка и ссылки

- Портал разработчика iHealth: `https://dev.ihealthlabs.com`
- Документация SDK: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/`
- Разделы по BP5S (Android/iOS): см. ссылки в начале файла.


