# VBotPhoneSDKAndroid-Public

- **Package:** `com.vpmedia.sdkvbot`
- **Điểm vào chính:** `com.vpmedia.sdkvbot.client.VBotClient`
- **minSdk:** 23 · **compileSdk:** 34 · **Ngôn ngữ:** Kotlin/Java
- **Phiên bản hiện tại:** `1.1.0`

## Cài đặt

### 1. Thêm repository

Thêm vào `settings.gradle` (hoặc `build.gradle` gốc). Repo Public phục vụ như Maven tĩnh qua `raw.githubusercontent.com`, không cần credential:

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url 'https://raw.githubusercontent.com/VBotDevTeam/VBotPhoneSDKAndroid-Public/main/' }
    }
}
```

### 2. Thêm SDK vào `app/build.gradle`

```groovy
dependencies {
    implementation 'com.github.VBotDevTeam:VBotPhoneSDKAndroid-Public:1.1.0'
}
```

## Quyền (permissions)

SDK khai sẵn 3 quyền: `DISABLE_KEYGUARD`, `POST_NOTIFICATIONS`, `USE_FULL_SCREEN_INTENT`. **App của bạn PHẢI tự khai** trong `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

Và xin **runtime permission** cho `RECORD_AUDIO` (bắt buộc để gọi/nghe) và `POST_NOTIFICATIONS` (Android 13+). Ví dụ dùng Activity Result API:

```kotlin
private val permissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { result ->
    val micGranted = result[Manifest.permission.RECORD_AUDIO] == true
    // micGranted == false → chặn luồng gọi, hướng dẫn user vào Settings bật lại
}

private fun requestCallPermissions() {
    val needed = mutableListOf(Manifest.permission.RECORD_AUDIO)
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        needed.add(Manifest.permission.POST_NOTIFICATIONS)
    }
    permissionLauncher.launch(needed.toTypedArray())
}
```

> Gọi `requestCallPermissions()` trước khi bắt đầu cuộc gọi (hoặc lúc khởi động app). Phải khai `RECORD_AUDIO` trong manifest trước thì lời gọi runtime mới hiện được dialog.

## Sử dụng

### 1. Khởi tạo

```kotlin
import com.vpmedia.sdkvbot.client.VBotClient
import com.vpmedia.sdkvbot.client.VBotConfig
import com.vpmedia.sdkvbot.client.VBotEnvironment

val client = VBotClient(applicationContext)

// Chọn môi trường: PRODUCTION (mặc định) | STAGING | SANDBOX
client.setup(VBotConfig(VBotEnvironment.PRODUCTION))

// Hoặc trỏ base URL tuỳ biến (ghi đè môi trường):
// client.setup(VBotConfig(customBaseUrl = "https://your-host/v3.0/"))
```

### 2. Đăng ký listener

Kế thừa `ClientListener` (open class), override phương thức cần dùng:

```kotlin
import com.vpmedia.sdkvbot.client.ClientListener
import com.vpmedia.sdkvbot.en.AccountRegistrationState
import com.vpmedia.sdkvbot.en.CallState
import com.vpmedia.sdkvbot.en.EndCallReason

private val listener = object : ClientListener() {
    override fun onUserConnected(displayName: String) {}
    override fun onCallState(state: CallState) {
        // Null, Calling, Incoming, Early, Connecting, Confirmed, Disconnected
    }
    override fun onCallEnded(reason: EndCallReason) { /* reason.code / reason.name */ }
    override fun onExternalCallId(externalCallId: String) { /* fire khi có cuộc gọi ĐẾN */ }
    override fun onCallMuteStateChanged(muted: Boolean) {}
    override fun onNetworkUnreachable() {}
    override fun onAccountRegistrationState(status: AccountRegistrationState, reason: String) {

    }
    override fun onErrorCode(erCode: Int, message: String) {}
}

client.addListener(listener)
// Nhớ gỡ khi huỷ:
client.removeListener(listener)
```

### 3. Kết nối tài khoản

`connect()`

```kotlin
client.connect(token = "vbot-token", tokenFirebase = "fcm-token") { displayName, error ->
    if (error == null) { /* ok — displayName */ } else { /* error.code / error.message */ }
}
```

### 4. Gọi đi (outgoing)

```kotlin
val externalCallId = generateId() // tối đa 32 ký tự [a-z0-9]
client.startOutgoingCall(hotline = "1900xxxx", phone = "0901234567", externalCallId = externalCallId) { _, error ->
    if (error != null) {
        // thất bại trước khi đổ chuông (vd: register timeout, AnotherCallInProgress)
    }
}
```

- `hotline` có thể để rỗng nếu không dùng đầu số.
- Completion `VBotCompletion<Unit>` là **tuỳ chọn** — có overload không completion.
- Diễn biến cuộc gọi theo dõi qua `onCallState` / `onCallEnded`.

### 5. Nhận cuộc gọi (incoming qua push)

Trong `FirebaseMessagingService` của bạn, chuyển payload sang SDK:

```kotlin
override fun onMessageReceived(message: RemoteMessage) {
    val map = HashMap(message.data) // cần chứa "transId" và "offCall"
    VBotClient(applicationContext).apply {
        if (!isSetup()) setup(VBotConfig(VBotEnvironment.PRODUCTION))
        notificationCall(map)
    }
}
```

- `offCall == "0"`: hiện notification cuộc gọi đến + chuẩn bị. Nếu không register được trong **20 giây** → `onCallEnded(IncomingCallTimeout)` + tự dọn.
- `offCall != "0"`: cuộc gọi đã bị huỷ từ xa → huỷ chuẩn bị, gỡ notification.

> SDK không khai `FirebaseMessagingService` — app của bạn sở hữu service và tự route payload noticall vào `notificationCall(map)`, nên không tranh chấp `MESSAGING_EVENT` với `firebase_messaging` sẵn có.

Trả lời / từ chối:

```kotlin
client.answerCall()                     // nghe máy
client.declineIncomingCall(isBusy = true) // từ chối (true = báo bận / Busy Here)
```

### 6. Điều khiển trong cuộc gọi

```kotlin
client.endCall()                 // kết thúc cuộc gọi hiện tại
client.muteCall(true)            // bật/tắt mic
client.isCallMute()              // trạng thái mic
client.onOffSpeaker(true)        // bật/tắt loa ngoài
client.isSpeakerOn()             // trạng thái loa
client.sendDTMF("123")           // gửi phím DTMF
client.getDuration()             // thời lượng cuộc gọi (giây) hoặc null
client.callName()                // remote URI hoặc null
client.hasActiveCall()           // đang có cuộc gọi không
```

### 7. Danh sách hotline

`getHotlines()` là `suspend` — gọi trong coroutine:

```kotlin
lifecycleScope.launch {
    val hotlines: List<Hotline>? = client.getHotlines()  // null → xem onErrorCode
}
```

### 8. Ngắt kết nối

`disconnect()`

```kotlin
client.disconnect { _, error ->
    // credential luôn được xoá cục bộ dù server logout lỗi hay không
}
```

## Java interop

- Completion là `fun interface VBotCompletion<T>` → gọi bằng lambda trong cả Kotlin và Java.
- `VBotError` có `getCode()` / `getMessage()`.
- Các overload cũ (`connect(token, push)` không completion, `disconnect(): Boolean`, `startCall(...)`, `isCall()`) được đánh dấu `@Deprecated` — nên dùng API mới.
