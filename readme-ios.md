### Интеграция iHealth SDK на iOS (BP5S)

Инструкция по добавлению iHealth SDK в iOS‑приложение и базовой работе с тонометром BP5S.

Ссылки:
- Quickstart: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/ios/quickstart`
- BP5S: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/ios/blood_pressure/bp5s`
- Загрузка SDK: `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/ios`

---

### 1) Требования

- Подтвержденная лицензия для точного **Bundle Identifier** вашего приложения.
- Xcode 14+ (рекомендуется), iOS 12+ (рекомендуется), Bluetooth LE.
- SDK‑файлы из портала или папки `sdk/ios` текущего репозитория.

---

### 2) Добавление SDK в проект

1. Добавьте в проект библиотеку iHealth SDK (из комплекта это могут быть `.framework`/`.a` и заголовки `.h`).
2. Убедитесь, что библиотеки попадают в Target → General → Frameworks, Libraries, and Embedded Content (Embed & Sign — для dynamic frameworks).
3. Поместите файл лицензии (например, `ihealth_license.pem`) в основной бандл приложения.

Info.plist (минимум):

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth is required to connect to iHealth devices.</string>
<key>UIBackgroundModes</key>
<array>
    <string>bluetooth-central</string>
    <!-- Добавьте external-accessory только для устройств по EA-протоколам -->
</array>
```

External Accessory (UISupportedExternalAccessoryProtocols) добавляйте только если используете устройства iHealth по EA (не актуально для BP5S, который BLE).

---

### 3) Валидация лицензии SDK

Выполните валидацию лицензии до начала сканирования/подключения.

```objective-c
#import <IHSDKCloudUser/IHSDKCloudUser.h>

- (void)validateSDKLicense {
    NSString *path = [[NSBundle mainBundle] pathForResource:@"ihealth_license" ofType:@"pem"];
    NSData *licenseData = [NSData dataWithContentsOfFile:path];
    if (!licenseData) {
        NSLog(@"License file not found");
        return;
    }
    [[IHSDKCloudUser commandGetSDKUserInstance] commandSDKUserValidationWithLicense:licenseData
                             UserDeviceAccess:^(NSArray *deviceArray) {
                                 // Available device types for this license
                                 NSLog(@"Devices: %@", deviceArray);
                             }
                       UserValidationSuccess:^{
                                 NSLog(@"SDK license validated");
                             }
                           DisposeErrorBlock:^(SDKUserValidationError errorID) {
                                 NSLog(@"SDK validation error: %ld", (long)errorID);
                             }];
}
```

Для проектов на Swift используйте Bridging Header либо обертку на Objective‑C.

---

### 4) Сканирование и подключение к BP5S

Подход с контроллерами сканирования и подключения:

```objective-c
#import <IHSDKDemo/IHSDKDemo.h> // Замените на актуальные заголовки из вашего SDK

- (void)startScanBP5S {
    // Subscribe to notifications (names may vary depending on SDK build)
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(deviceFound:) name:BP5SDiscover object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(deviceConnectFailed:) name:BP5SConnectFailed object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(deviceConnected:) name:BP5SConnectNoti object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(deviceDisconnected:) name:BP5SDisConnectNoti object:nil];

    // Start scanning
    [[ScanDeviceController commandGetInstance] commandScanDeviceType:HealthDeviceType_BP5S];
}

- (void)connectToBP5S:(NSString *)serialOrMac {
    [[ConnectDeviceController commandGetInstance] commandContectDeviceWithDeviceType:HealthDeviceType_BP5S andSerialNub:serialOrMac];
}

- (void)deviceFound:(NSNotification *)noti {
    // Extract device identifier from noti.userInfo and connect
}

- (void)deviceConnected:(NSNotification *)noti {
    // Ready to interact with BP5S instance
}
```

Альтернативно используйте соответствующий контроллер устройства (например, `BP5SController`), если он предоставлен вашей версией SDK, и его API для подписки на события и команд.

---

### 5) Пример получения заряда батареи и базовых команд

Методы и сигнатуры могут отличаться между версиями SDK. Ниже — иллюстративный пример для получения уровня заряда и статуса:

```objective-c
// Suppose bp5s is a connected device instance obtained from the SDK
[bp5s commandEnergy:^(NSNumber *energyValue) {
    NSLog(@"Battery: %@", energyValue);
} energyState:^(NSNumber *energyState) {
    NSLog(@"Battery state: %@", energyState);
} errorBlock:^(BP5SError errorID) {
    NSLog(@"BP5S error: %ld", (long)errorID);
}];

// Start measurement (refer to BP5S documentation for the exact API available in your SDK build)
// [bp5s commandStartMeasureWith...];
```

Результаты измерений и другие события обрабатывайте через предоставленные SDK блоки/делегаты/уведомления — см. официальную документацию BP5S для вашей версии.

---

### 6) Завершение работы

- Остановите сканирование при ненужности:

```objective-c
[[ScanDeviceController commandGetInstance] commandStopScan];
```

- Отпишитесь от уведомлений и корректно разорвите соединение с устройством. Способ зависит от используемых контроллеров в вашей версии SDK.

---

### 7) Частые проблемы

- Лицензия не валидируется: проверьте точное совпадение `Bundle Identifier` и статус приложения на портале.
- Нет обнаружения устройств: убедитесь в наличии Bluetooth, разрешении iOS на использование Bluetooth и включенном `bluetooth-central` (если требуется фон).
- Отличаются имена уведомлений/методов: используйте актуальные заголовки и справочник вашей версии SDK (см. ссылки выше).


