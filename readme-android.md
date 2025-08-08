### Интеграция iHealth SDK на Android (BP5S)

Ниже — пошаговая инструкция по подключению iHealth SDK и работе с тонометром BP5S в Android‑приложении.

Ссылки:
- Quickstart: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/android/quickstart/`
- BP5S: `https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/docs/android/blood_pressure/bp5s`
- Загрузка SDK: `https://dev.ihealthlabs.com/workspace/solution/version/NativeSDK/main/sdk/android`

---

### 1) Требования

- Подтвержденная лицензия для точного `Package Name` (ваш `applicationId`).
- Android 8.0+ (рекомендуется), Bluetooth LE.
- SDK‑файлы из портала или папки `sdk/android` текущего репозитория.

---

### 2) Добавление SDK в проект

1. Поместите библиотечные файлы из SDK в проект:
   - `.jar`/`.aar` — в `app/libs`
   - `.so` — в `app/src/main/jniLibs/<abi>` (или скопируйте как есть из комплекта SDK)
2. В `build.gradle` модуля укажите путь к `jniLibs` (если требуется):

```groovy
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs', 'src/main/jniLibs']
        }
    }
}
```

3. Добавьте файл лицензии в `app/src/main/assets` (например, `ihealth_license.pem`).

---

### 3) Разрешения и манифест

Добавьте в `AndroidManifest.xml` необходимые Bluetooth‑разрешения. Для совместимости укажите разные наборы по версиям Android:

```xml
<!-- Android 12+ (API 31+) -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />

<!-- Android 10–11 (API 29–30) для BLE‑сканирования используется геолокация -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- Обязательно: включить BLE в манифесте устройства -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true" />
```

На Android 6.0+ не забудьте запрашивать runtime‑разрешения.

---

### 4) Инициализация SDK и валидация лицензии

Инициализируйте SDK в вашем `Application` и выполните валидацию лицензии до начала работы с устройствами.

```java
// Application class
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        iHealthDevicesManager.getInstance().init(this, Log.VERBOSE, Log.VERBOSE);
        authenticateSdk();
    }

    private void authenticateSdk() {
        try (InputStream is = getAssets().open("ihealth_license.pem")) {
            byte[] buffer = new byte[is.available()];
            // Read license into memory
            int read = is.read(buffer);
            if (read <= 0) throw new IOException("Empty license file");
            boolean ok = iHealthDevicesManager.getInstance().sdkAuthWithLicense(buffer);
            if (!ok) {
                // Handle license validation failure
                Log.e("iHealth", "SDK license validation failed");
            }
        } catch (IOException e) {
            Log.e("iHealth", "License load error", e);
        }
    }
}
```

---

### 5) Сканирование, подключение и базовые операции (BP5S)

Зарегистрируйте колбэк, отфильтруйте события по типу устройства, выполните сканирование и подключение. Затем получите `Bp5sControl` для запросов.

```java
// Register callback
private final iHealthDevicesCallback callback = new iHealthDevicesCallback() {
    @Override
    public void onScanDevice(String mac, String deviceType, int rssi, Map manufactorData) {
        // Device discovered
    }

    @Override
    public void onDeviceConnectionStateChange(String mac, String deviceType, int status, int errorID, Map manufactorData) {
        // Connection state changed
    }

    @Override
    public void onDeviceNotify(String mac, String deviceType, String action, String message) {
        // Handle device notifications (battery, function info, measurement results)
        if (BpProfile.ACTION_BATTERY_BP.equals(action)) {
            try {
                JSONObject obj = new JSONObject(message);
                int battery = obj.getInt(BpProfile.BATTERY_BP);
                int status = obj.getInt(BpProfile.BATTERY_STATUS);
                Log.d("iHealth", "Battery=" + battery + ", status=" + status);
            } catch (JSONException e) {
                Log.e("iHealth", "Battery parse error", e);
            }
        }
        // Check BpProfile constants for other actions/keys (see official docs)
    }
};

int cbId = iHealthDevicesManager.getInstance().registerClientCallback(callback);
iHealthDevicesManager.getInstance().addCallbackFilterForDeviceType(cbId, iHealthDevicesManager.TYPE_BP5S);

// Start scanning
iHealthDevicesManager.getInstance().startDiscovery(DiscoveryTypeEnum.BP5S);

// After you picked a MAC address from discovery
String mac = "XX:XX:XX:XX:XX:XX"; // Replace with actual MAC
iHealthDevicesManager.getInstance().connectDevice("", mac, iHealthDevicesManager.TYPE_BP5S);

// After connected
Bp5sControl control = iHealthDevicesManager.getInstance().getBp5sControl(mac);
if (control != null) {
    control.getBattery();
    control.getFunctionInfo();
    // Start measurement (refer to docs for exact flow & available options)
    // control.startMeasure();
}
```

Обработку результатов измерения выполняйте в `onDeviceNotify`, используя профильные константы `BpProfile.*` (см. документацию BP5S). Ключи JSON и события зависят от версии SDK.

---

### 6) Завершение работы

- Остановите сканирование при ненадобности: 

```java
iHealthDevicesManager.getInstance().stopDiscovery();
```

- При закрытии экрана/приложения снимите колбэк и при необходимости отключитесь от устройства:

```java
iHealthDevicesManager.getInstance().unRegisterClientCallback(cbId);
// iHealthDevicesManager.getInstance().disconnectDevice(mac);
```

---

### 7) Частые проблемы

- Лицензия не проходит валидацию: проверьте точное совпадение `applicationId` и имя лицензии/приложения на портале.
- Не находится устройство: включите Bluetooth и геолокацию (для Android 10–11), проверьте runtime‑разрешения, заряд устройства.
- Нет нотификаций: проверьте регистрацию колбэка и фильтрацию по типу BP5S.


