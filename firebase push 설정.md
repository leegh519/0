# Flutter Push 적용법
김승연 2022.12.20 작성
<br></br>
## 1. 플러터 코드 수정
---
1. pubspec.yaml에 다음 추가(버전은 pub.dev 기반)
```yaml
  firebase_core: ^2.4.0
  firebase_messaging: ^14.1.4
  flutter_local_notifications: ^13.0.0
```

2. /resoureces/constants.dart에 아래 내용 추가
```dart
const AndroidNotificationDetails androidPlatformChannelSpecifics =
    AndroidNotificationDetails('kr.co.brighten.saveuson', '세이버스ON',
        channelDescription: '세이버스ON',
        playSound: true,
        enableVibration: true,
        icon: null,
        importance: Importance.max,
        priority: Priority.high);
const DarwinNotificationDetails iOSPlatformChannelSpecifics =
    DarwinNotificationDetails();
const NotificationDetails platformChannelSpecifics = NotificationDetails(
    android: androidPlatformChannelSpecifics, iOS: iOSPlatformChannelSpecifics);
```
3. /resoureces/.에 push_setting.dart 생성
```dart
import 'package:saveuson/resoureces/constants.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter/material.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

const androidNotificationChannel = AndroidNotificationChannel(
    'kr.co.brighten.saveuson', '세이버스ON',
    description: '세이버스ON 알림', importance: Importance.max);

String? appToken;
final FirebaseMessaging firebaseMessaging = FirebaseMessaging.instance;
final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
    FlutterLocalNotificationsPlugin();

void registerNotification() {
  firebaseMessaging.requestPermission();

  FirebaseMessaging.onMessage.listen((RemoteMessage message) {
    if (message.notification != null) {
      showNotification(message.notification!);
    }
    return;
  });

  // FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);

  firebaseMessaging.getToken().then((token) async {
    await FirebaseMessaging.instance.setAutoInitEnabled(true);
    appToken = token;
    debugPrint(appToken);
  });
}

void showNotification(RemoteNotification remoteNotification) async {
  await flutterLocalNotificationsPlugin.show(0, remoteNotification.title,
      remoteNotification.body, platformChannelSpecifics,
      payload: null);
}

void configLocalNotification() async {
  AndroidInitializationSettings initializationSettingsAndroid =
      const AndroidInitializationSettings('@mipmap/ic_launcher');
  DarwinInitializationSettings initializationSettingsIOS =
      const DarwinInitializationSettings();
  InitializationSettings initializationSettings = InitializationSettings(
      android: initializationSettingsAndroid, iOS: initializationSettingsIOS);
  await flutterLocalNotificationsPlugin
      .resolvePlatformSpecificImplementation<
          AndroidFlutterLocalNotificationsPlugin>()
      ?.createNotificationChannel(androidNotificationChannel);
  flutterLocalNotificationsPlugin.initialize(initializationSettings);
}

```
4. main.dart에 아래 코드 추가
```dart
void main() async {

  ...

  await Firebase.initializeApp().then((value) async {
    registerNotification();
    configLocalNotification();
  });

  ...

  runApp(const Saveuson());
}
```

5. ios/podfile 다음으로 수정
```Pod
# Uncomment this line to define a global platform for your project
platform :ios, '12.0' # 이거

# CocoaPods analytics sends network stats synchronously affecting flutter build latency.
ENV['COCOAPODS_DISABLE_STATS'] = 'true'

project 'Runner', {
  'Debug' => :debug,
  'Profile' => :release,
  'Release' => :release,
}

def flutter_root
  generated_xcode_build_settings_path = File.expand_path(File.join('..', 'Flutter', 'Generated.xcconfig'), __FILE__)
  unless File.exist?(generated_xcode_build_settings_path)
    raise "#{generated_xcode_build_settings_path} must exist. If you're running pod install manually, make sure flutter pub get is executed first"
  end

  File.foreach(generated_xcode_build_settings_path) do |line|
    matches = line.match(/FLUTTER_ROOT\=(.*)/)
    return matches[1].strip if matches
  end
  raise "FLUTTER_ROOT not found in #{generated_xcode_build_settings_path}. Try deleting Generated.xcconfig, then run flutter pub get"
end

require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)

flutter_ios_podfile_setup

target 'Runner' do
  use_frameworks!
  use_modular_headers!

  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
end

# target 'ImageNotification' do
#   use_frameworks!
#   pod 'Firebase/Messaging'
# end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '12.0' # 이거
    end
  end
end
```
<br></br>
## 2. 파이어베이스 기본세팅

> https://console.firebase.google.com/ 에서 파이어베이스 프로젝트 생성
---
2. 프로젝트 이름 지정
3. 계속(Google 애널리틱스 사용여부 상관없음) 
4. default Account for Firebase

<br></br>
## 3. 안드로이드 등록
> 앱에 Firebase를 추가하여 시작하기에 안드로이드 클릭
---
1. 앱등록  
- /android/app/build.gradle 에 있는 applicationId로 입력  
- 선택사항 넘기기

2. 구성파일 다운로드 후 추가  
- 다운로드한 google-services.json 파일을
모듈(앱 수준) 루트 디렉터리(/android/app/.)로 이동
3. 코드 추가 
- /android/build.gradle에 다음 사항 추가
```gradle
 dependencies {
        ...
        classpath 'com.google.gms:google-services:4.3.13' // << 추가 (firebase 가이드하는 버전으로 맞추기)
    }
```
- /android/app/build.gradle에 다음 사항 추가
    > dependencies 없을 경우 apply from 밑에 추가
```gradle
apply plugin: 'com.android.application' // << 추가
...
apply plugin: 'com.google.gms.google-services' // << 추가
... 
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

dependencies {
  implementation platform('com.google.firebase:firebase-bom:31.1.1') // << 추가 (firebase 가이드하는 버전으로 맞추기)
  implementation 'com.google.firebase:firebase-analytics' // << 추가 
} 
```
<br></br>

## 4. ios 등록
---
1. 앱등록  
- 번들 ID는 Xcode의 앱 기본 대상에 대한 일반 탭에서 확인

2. 구성 파일 다운로드
- GoogleService-Info.plist를 /ios/Runner/. 에 추가  
- Xcode 열고 파일 > Add files to Runner > GoogleService-Info.plist 추가  
- Xcode에서 Runner 디렉토리 밑에 추가 되었는지 확인

3. Firebase SDk 추가(없어도 동작함 - 에러 발생시 제거 또는 추가 하지 말것)
- Xcode에서 File > Add Packages > https://github.com/firebase/firebase-ios-sdk 검색
- firebase-ios-sdk 클릭  
- Add to Project Runner 로 지정
- Add Packages
- 추가 완료 후 Xcode에서 FirebaseAnalytics 추가

> 아래 모든 작업까지 끝나고 최종 실행 시 에러 날 경우  
> Xcode Runner 클릭 project Runner 클릭 package dependencies 에서 sdk 제거

4. 초기화 코드 추가  
/ios/Runner/AppDelegate.swift 에 해당 코드로 변경
```swift
import UIKit
import Flutter
import FirebaseCore

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    FirebaseApp.configure()
    GeneratedPluginRegistrant.register(with: self)
    if #available(iOS 12.0, *) { // 이거 버전 맞추기
    UNUserNotificationCenter.current().delegate = self as? UNUserNotificationCenterDelegate
    }
    application.registerForRemoteNotifications()
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

5. Xcode에서 Signing & Capabilities 설정 추가
> team 체크(BRIGHTEN)

- **+Capability**  클릭
    - Push Notifications 추가
    - Background Modes 추가
        - Background fetch 체크
        - Remote notifications 체크

6. 인증서 키체인으로 발급
> 맥 런치패드 > 키체인 접근
- 상단 키체인 접근 > 인증서 지원 > 인증기관에서 인증서 요청
- 본인 애플 개발자 계정 쓰기 
- 요청항목 디스크에 저장됨 클릭
- 계속
- 해당 파일위치 기억해두기 (7번에서 쓰임)


7. 애플 developer 인증서 추가
> https://developer.apple.com/kr/ 개발자 계정 접속  
> Certificates, Identifiers & Profiles(인증서, 식별자 및 프로파일) > 식별자(영문)로 이동

- Identifiers 목록 하단에서 내가 등록한 앱 찾아서 클릭
- push Notification (configure) 클릭
- Development SSL Certificate 은 개발용  
    - create Certificate 클릭
    - choose file 에서 키체인으로 생성한 인증서 추가
    - 다운로드 후 따로 폴더 만든 후 보관 (추후 사용될 수도 있음)
- Production SSL Certificate 은 배포용
    - 개발용과 동일하게 진행

8. firebase에 인증서 추가
 - 7번에서 발급받은 인증서 p12 파일로 변환 (개발용, 배포용 둘다 변환하기)
    - 7번에서 발급받은 인증서를 더블클릭 후 키체인 접근에 추가하기  
    - 키체인 접근에서 해당 인증서 오른쪽 클릭 후 내보내기
    - p12 파일로 내보내기
    - 암호 설정 안하고 다음 시스템 비밀번호 입력 후 완료
- APN 인증서 추가 (개발용, 배포용)
    > 위치 firebase > ios 프로젝트 설정 > 클라우드 메시징


9. ios release 모드 일때 안될 수 있으므로 /ios/info.plist에 아래 추가
```plist
<key>FirebaseAppDelegateProxyEnabled</key>
<string>NO</string>
```
10. flutter clean 및 flutter pub get


<br></br>
## 5. 테스트
---
> firebase > 참여 messaging > 첫 번째 캠페인 만들기 > 알림 메세지 만들기
1. 제목 입력
2. 알림 텍스트 입력
3. 테스트 메세지 전송
4. FCM 등록 및 토큰 추가에 앱 토큰 추가(앱시작시 디버그콘솔에 출력)
5. 테스트


