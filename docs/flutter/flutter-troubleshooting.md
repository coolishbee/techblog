---
comments: true
---

## Flutter 버전 마이그레이션 이슈

에러메시지
```
Your Flutter application is created using an older version of the Android embedding.
```

`AndroidManifest.xml` 안에 `activity`태그에 추가해주세요.
```
<meta-data
         android:name="flutterEmbedding"
         android:value="2" />
```

!!! 출처
    [https://stackoverflow.com/questions/64425132/how-to-fix-flutter-warning-your-flutter-application-is-created-using-an-older-v](https://stackoverflow.com/questions/64425132/how-to-fix-flutter-warning-your-flutter-application-is-created-using-an-older-v)


## Android license 해결 안되는 이슈

`flutter doctor` 시에 뜨는 android license
```
flutter doctor --android-licenses
```

sdkmanager 관련 오류메시지
```
Android sdkmanager tool was found, but failed to run...
```

안드로이드 스튜디오에서 SDK Command-line Tools 를 설치해야 함.
그러나 최신버전(13)은 안되고 버전 10으로 설치하니깐 해결되었다.

!!! 출처
    [https://stackoverflow.com/questions/76882205/error-linkageerror-occurred-while-loading-main-class-com-android-sdklib-tool-sd](https://stackoverflow.com/questions/76882205/error-linkageerror-occurred-while-loading-main-class-com-android-sdklib-tool-sd)

!!! 출처
    [https://stackoverflow.com/questions/75103753/android-sdkmanager-tool-found-but-failed-to-run-for-flutter-doctor-android](https://stackoverflow.com/questions/75103753/android-sdkmanager-tool-found-but-failed-to-run-for-flutter-doctor-android)

## RenderObject 사용시 마이그레이션 이슈

에러메시지
```
A value of type 'RenderObject' can't be assigned to a variable of type 'RenderBox'
```

```dart
final RenderBox renderBox = scheduleForDayKey.currentContext!
                                             .findRenderObject() as RenderBox;
```

!!! 출처
    [https://stackoverflow.com/questions/67550315/a-value-of-type-renderobject-cant-be-assigned-to-a-variable-of-type-renderbo](https://stackoverflow.com/questions/67550315/a-value-of-type-renderobject-cant-be-assigned-to-a-variable-of-type-renderbo)