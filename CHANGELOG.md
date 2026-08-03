# Changelog

Các thay đổi của VBot Phone SDK Android.

## 1.1.2

_Ngày phát hành: 03/08/2026_

### Tính năng mới

- Callback mới `onCallEnded(reason: VBotEndCallReason, endedBy: VBotCallEndParty)` cho biết nguyên nhân và bên kết thúc cuộc gọi. Callback một tham số cũ vẫn tương thích ngược.

### VBotEndCallReason mới

| reason                     | SIP |
| -------------------------- | --- |
| `incorrectInformation`     | 400 |
| `unauthenticated`          | 401 |
| `insufficientBalance`      | 402 |
| `recipientBlocksCalls`     | 403 |
| `destinationNotFound`      | 404 |
| `callIntervalNotAllowed`   | 405 |
| `memberNotActivated`       | 406 |
| `memberNotInProject`       | 407 |
| `doNotDisturb`             | 409 |
| `destinationGone`          | 410 |
| `recipientAbsent`          | 411 |
| `packageExpired`           | 412 |
| `hotlineTelcoNotSupported` | 413 |
| `telcoNotFound`            | 414 |
| `invalidParameter`         | 415 |
| `projectExpired`           | 416 |
| `callerCanceled`           | 487 |
| `connectionError`          | 500 |
| `transmissionError`        | 502 |

### Cập nhật dependency

```groovy
implementation 'com.github.VBotDevTeam:VBotPhoneSDKAndroid-Public:1.1.2'
```

## 1.1.1

_Ngày phát hành: 30/07/2026_

### Thay đổi phá vỡ tương thích

- **`EndCallReason` đổi tên thành `VBotEndCallReason`**, và toàn bộ 23 case chuyển sang camelCase để khớp iOS SDK: `Normal` → `normaly`, `Busy` → `busy`, `Unknown` → `unknownError`, `TimeOut` → `timeOut`… App đang `when(reason)` theo tên cũ cần đổi theo tên mới.

Sau bản này, tên case và mã số của `VBotEndCallReason` giống hệt iOS SDK, nên logic xử lý mã lỗi dùng chung được cho cả hai nền tảng.

### Cập nhật

- `VBotEndCallReason` có thêm thuộc tính `description` — mô tả lỗi dạng câu, thay cho việc đọc `.name`.
- `onErrorCode(code, message)` và `VBotError.message` giờ trả mô tả đọc được thay vì tên hằng số (ví dụ `"No response from server"` thay cho `"DataEmpty"`).
- `VBotError` có thêm factory `VBotError.from(reason)` và `VBotError.api(code, description)`.

### Cập nhật dependency

```groovy
implementation 'com.github.VBotDevTeam:VBotPhoneSDKAndroid-Public:1.1.1'
```

## 1.1.0

_Ngày phát hành: 26/07/2026_

### Tính năng mới

- **Thin/Shaded AAR — tích hợp một dòng, hết xung đột thư viện:** toàn bộ thư viện bên thứ ba (retrofit, okhttp, okio, gson, rxjava, Java-WebSocket, slf4j…) được đóng gói sẵn trong SDK dưới namespace riêng (`com.vpmedia.sdkvbot.shaded.*`). Chỉ cần khai một dòng dependency, không cần khai lại thư viện nào, không còn lỗi "Duplicate class" hay bị ép hạ cấp okio/okhttp.
- **Callback `onCallEnded(reason)`** trả mã kết thúc cuộc gọi, và **`onExternalCallId`** fire khi có cuộc gọi đến.

### Cập nhật

- **Chuyển repository sang `raw.githubusercontent.com`** (Maven tĩnh, không cần credential), thay cho `jitpack.io`.
- `disconnect()` dùng completion `VBotCompletion` thay cho `onSuccess`/`onFailure`.
- `startOutgoingCall(hotline, phone, externalCallId, completion)` là API gọi đi chính thức; `startCall(...)`, `connect(token, push)` không completion, `disconnect(): Boolean`, `isCall()` được đánh dấu `@Deprecated`.
- SDK **không** tự khai `FirebaseMessagingService` — app sở hữu service và route payload noticall vào `notificationCall(map)`, không tranh chấp `MESSAGING_EVENT` với `firebase_messaging` sẵn có.
- Code nội bộ được obfuscate; consumer ProGuard rules kèm sẵn — app không cần khai thêm rule nào kể cả khi bật minify.

### Nâng cấp từ 1.0.x

Xoá toàn bộ các dòng khai báo rxjava / gson / retrofit / okhttp / reactive-streams / timber mà tài liệu cũ yêu cầu, cùng mọi `exclude group: 'com.squareup.okio'`. Đổi repository từ `jitpack.io` sang `raw.githubusercontent.com`.

## 1.0.12

_Ngày phát hành: 06/07/2026_

### Tính năng mới

- **Cấu hình môi trường API & custom base URL:** thêm `VBotConfig` và enum `VBotEnvironment` (PRODUCTION, STAGING, SANDBOX), hoặc trỏ API URL tuỳ biến qua `customBaseUrl` khi gọi `setup(config)`.

## 1.0.11

_Ngày phát hành: 13/02/2026_

### Tính năng mới

- `startOutgoingCall` hỗ trợ tham số `externalCallId` (tuỳ chọn), cho phép truyền mã cuộc gọi từ hệ thống bên ngoài để liên kết dữ liệu.
