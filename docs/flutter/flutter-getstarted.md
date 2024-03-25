---
comments: true
---

## Flutter 개발환경 설치 for Mac
### 1. Flutter SDK 설치

Flutter를 사용하기 위해서는 Flutter SDK를 설치해야 합니다.

!!! info

    [다운로드 및 가이드 사이트](https://flutter.dev/docs/get-started/install/macos)로 이동하여 설치를 진행해주세요.

압축을 풀고 적당한 위치에 폴더를 구성해준다

```
~/Users/test/SDK/flutter
```

![finder](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fzpi3r%2FbtqFVjeKqgX%2FTKeNaKdbompouOYiZt7VE1%2Fimg.png)

### 2. 환경변수 등록

핵심은 환경변수 셋팅이라고 생각한다.

bash 나 zsh 나 똑같다.(참고로 필자는 zsh 사용중)

```
export FLUTTER_BIN="/Users/test/SDK/flutter/bin"
export PATH=$PATH:$FLUTTER_BIN
```

각자 다른 개발환경변수들도 셋팅 되어있기때문에 플루터변수도 따로 설정해줬다.

이부분을 잘 모른다면 따로 규칙을 찾아보시길...

### 4. 개발환경 셋팅 확인

커맨드 명령어로 제대로 셋팅이 되었는지 체크해볼 수 있다.

```
flutter --version
```

버전확인

```
flutter doctor
```

플러터 개발에 필요한 다트, 안드로이드, iOS 관련된 툴 및 SDK 설치 점검

만약에 flutter 명령어가 안먹는다면 환경변수 셋팅문제다.

```
which flutter
which dart
```

했을때 설치경로가 나오니깐 참고하고 정상적으로 설정되었는지 체크해본다.

### 5. Dart 설치

이건 간단해서 공식사이트만 참고해도 충분히 어렵지 않다.

!!! note

    [다트 공식사이트](https://dart.dev/get-dart). (단, brew 가 설치되어있어야 한다)

```
brew tap dart-lang/dart
brew install dart
```

다트 설치

```
brew upgrade dart
```

다트 최신버전 업그레이드

```
brew switch dart 2.1.0
```

다트 버전 스위칭

```
brew info dart
```

다트 버전 정보 출력

## flutter 명령어

[flutter command-line tool](https://docs.flutter.dev/reference/flutter-cli) 는 플러터와 상호 작용하는 방식이다.
[dart](https://dart.dev/tools/dart-tool) cli 관련 문서는 위 경로에서 확인할 수 있다.

- flutter 로 프로젝트 최초 생성시 패키지명 설정하기(마지막 . 생략)

```zsh
flutter create --org com.yourdomain appname //com.yourdomain.appname
//ex) flutter create --org com appname //com.appname
```

- [change_app_package_name](https://pub.dev/packages/change_app_package_name)를 이용하여 이미 생성된 프로젝트 패키지명 변경하기

```
flutter pub run change_app_package_name:main com.package.appname
```

!!! 출처
    [https://stackoverflow.com/questions/51534616/how-to-change-package-name-in-flutter](https://stackoverflow.com/questions/51534616/how-to-change-package-name-in-flutter)


## flutter 버전코드 버전네임 변경하기

`local.properties`에서 값을 변경해줘봤자 리셋된다. `pubspec.yaml`에서 바꿔줘야 한다.

- version: 1.0.1+2
- version name: 1.0.1
- version code: 2

??? info "pubspec.yaml"
    ```
    name: flutter_lineup_builder
    description: "A new Flutter project."
    # The following line prevents the package from being accidentally published to
    # pub.dev using `flutter pub publish`. This is preferred for private packages.
    publish_to: 'none' # Remove this line if you wish to publish to pub.dev

    # The following defines the version and build number for your application.
    # A version number is three numbers separated by dots, like 1.2.43
    # followed by an optional build number separated by a +.
    # Both the version and the builder number may be overridden in flutter
    # build by specifying --build-name and --build-number, respectively.
    # In Android, build-name is used as versionName while build-number used as versionCode.
    # Read more about Android versioning at https://developer.android.com/studio/publish/versioning
    # In iOS, build-name is used as CFBundleShortVersionString while build-number is used as CFBundleVersion.
    # Read more about iOS versioning at
    # https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html
    # In Windows, build-name is used as the major, minor, and patch parts
    # of the product and file versions while build-number is used as the build suffix.
    version: 1.0.1+2

    environment:
    sdk: '>=3.2.3 <4.0.0'
    ```

## 안드로이드 무선 디버깅 설정

```
adb connect [phone_ip]:[port]
```

!!! 출처
    [https://stackoverflow.com/questions/71353838/pair-new-device-over-wi-fi-not-working-in-android-studio-bumblebee](https://stackoverflow.com/questions/71353838/pair-new-device-over-wi-fi-not-working-in-android-studio-bumblebee)

## 안드로이드 배포준비하기

https://flutter-ko.dev/deployment/android