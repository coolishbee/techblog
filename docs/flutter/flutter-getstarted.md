# Flutter 개발환경 설치 for Mac

## 1. Flutter SDK 설치

Flutter를 사용하기 위해서는 Flutter SDK를 설치해야 합니다.

!!! info

    [다운로드 및 가이드 사이트](https://flutter.dev/docs/get-started/install/macos)로 이동하여 설치를 진행해주세요.

압축을 풀고 적당한 위치에 폴더를 구성해준다

```
~/Users/test/SDK/flutter
```

![finder](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fzpi3r%2FbtqFVjeKqgX%2FTKeNaKdbompouOYiZt7VE1%2Fimg.png)

## 2. 환경변수 등록

핵심은 환경변수 셋팅이라고 생각한다.

bash 나 zsh 나 똑같다.(참고로 필자는 zsh 사용중)

```
export FLUTTER_BIN="/Users/test/SDK/flutter/bin"
export PATH=$PATH:$FLUTTER_BIN
```

각자 다른 개발환경변수들도 셋팅 되어있기때문에 플루터변수도 따로 설정해줬다.

이부분을 잘 모른다면 따로 규칙을 찾아보시길...

## 4. 개발환경 셋팅 확인

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

## 5. Dart 설치

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