# Android SDK

---

## Yêu cầu

Android SDK version 23 trở lên

---

## Cấu hình SDK

### Cách 1

Trong file **app** → **build.gradle** thêm 

Cần thêm các thư viện cần thiết để SDK hoạt động 

```kotlin
dependencies {
		//các thư viện cần thiết để SDK hoạt động
		implementation("io.reactivex.rxjava2:rxjava:2.2.21")
    implementation("com.google.code.gson:gson:2.11.0")
    implementation("com.squareup.retrofit2:adapter-rxjava2:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("org.reactivestreams:reactive-streams:1.0.4")
    implementation ("com.squareup.okhttp3:okhttp:5.0.0-alpha.14")
    implementation("com.jakewharton.timber:timber:5.0.1")
    implementation("com.squareup.okhttp3:okhttp-dnsoverhttps:4.9.0")
    
		//Thêm SDK
    implementation 'com.github.VBotDevTeam:VBotPhoneSDKAndroid-Public:1.0.7'
}
```
Trong file **settings.gradle** thêm 
```kotlin
dependencyResolutionManagement {
		repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
		repositories {
			mavenCentral()
			maven { url 'https://jitpack.io' }
		}
	}
```
### Cách 2
- Vào trang https://jitpack.io/
- Trong vào ô *'Git repo url'* nhập **VBotDevTeam/VBotPhoneSDKAndroid-Public**
- Nhấn **Look up**
- Chọn version và nhấn **Get it**
- Làm theo hướng dẫn trên trang web
---

## Quyền

Danh sách các quyền cần thiết để SDK hoạt động:

1.	**Quyền truy cập điện thoại**

•	Kiểm tra trạng thái thiết bị (cuộc gọi đến/đi, đang bận, hoặc rảnh).

•	Thực hiện và quản lý cuộc gọi VoIP.

•	**Lưu ý**: Đây là quyền **bắt buộc** để SDK hoạt động.

2.	**Quyền truy cập micro** 

•	Thu âm giọng nói để gửi trong các cuộc gọi VoIP.

•	**Lưu ý**: Đây là quyền **bắt buộc** để SDK hoạt động.

3.	**Quyền truy cập thiết bị ở gần** 

•	Kết nối với các thiết bị Bluetooth như tai nghe hoặc loa ngoài để sử dụng trong cuộc gọi.

4.	**Quyền thông báo** 

•	Hiển thị thông báo cuộc gọi đến hoặc các sự kiện quan trọng từ SDK.

5.	**Quyền hiển thị trên ứng dụng khác** 

•	Hiển thị thông báo quan trọng (thông báo cuộc gọi đến) dưới dạng “màn hình nổi” hoặc “pop-up” ngay cả khi ứng dụng đang ở chế độ nền hoặc màn hình khóa.

6.	**Quyền cho phép sử dụng dữ liệu không hạn chế** 

•	Khi hiển thị thông báo cuộc gọi thì sdk có quyền này sẽ hoạt động tốt hơn, kết nối với cuộc gọi nhanh hơn
