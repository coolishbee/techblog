---
comments: true
---

## XCFramework 와 Fat Framework(Universal) 의 차이

### Fat(Universal) Framework

여러 아키텍쳐들의 형태로 빌드된 라이브러리들을 하나로 통합한 형태의 프레임워크가 Fat(Universal) Framework 이며
특징은 명령어로 특정 아키텍쳐를 삭제할 수 도 있고 원하는 아키텍쳐들을 합칠 수 도 있다.
만약 Fat Framework 를 사용하려면 아래와 같이 앱스토어 업로드시에는 시뮬레이터 아키텍쳐들을 제거해야 한다.

- Fat(Universal) Framework 는 다양한 아키텍쳐를 포함할 수 있다.
- 앱스토어에 업로드시 시뮬레이터용 아키텍쳐가 포함되지 않아야 한다.(strip 과정이라고 부름)

### XCFramework

실리콘 기반의 m1이 출시되면서 arm64 아키텍쳐가 iOS와 서로 겹치기 때문에 하나의 fat 파일로 만들지 못하는 문제를 해결하기 위해
멀티 아키텍쳐와 멀티플랫폼을 지원하도록 나온 새로운 프레임워크 개념.

- XCFramework 는 멀티 아키텍쳐나 플랫폼을 하나로 묶어서 배포할 수 있다.
- Xcode11 부터 생긴 새로운 배포방식.

```
    Fat(Universal)              appstore upload
|-------------------|       |-------------------|
|   i386            |       |                   |
|   x86_64          |       |                   |
|   armv7           |   ->  |   armv7           |
|   armv7s          |       |   armv7s          |
|   arm64           |       |   arm64           |
|-------------------|       |-------------------|

                    .xcframework
|---------------------------------------------------------------|
|   ios-arm64   ios-x86_64-simulator    ios-x86_64-maccatalyst  |
|   |-------|       |--------|              |--------|          |
|   | arm64 |       | x86_64 |              | x86_64 |          |
|   |-------|       |--------|              |--------|          |
|---------------------------------------------------------------|
```

```
포함된 아키텍쳐 확인하는 터미널 명령어
$ lipo -info libFileName.a

// Fat(Universal) Library 만들기
$ lipo -create -output libFileName.a libFileName-i386.a libFileName-x86_64.a libFileName-armv7.a libFileName-arm64.a

// 특정 아키텍쳐 삭제 (i386 아키텍쳐만 걸러내기)
$ lipo -remove i386 -output libFileNameOutput.a libFileName.a
```


## Library

### Static Library

* 파일확장자 `.a` 이며 리눅스에서 쓰이는 포맷이고 안드로이드에서도 사용가능한 확장자

* 앱의 컴파일 시점에 링크됨

* 이미지와 같은 리소스 파일들을 포함할 수 없기 때문에 Bundle 파일을 포함하여 배포 필요

### Dynamic Library

* 파일확장자 `.dylib` 이며 리눅스나 안드로이드에서는 `.so` 가 동일한 동적타입의 라이브러리이다.(c# dll)

* 앱의 런타임 시점에 링크되어 실행됨


## Framework

### Static Framework

전체소스를 제공하지 않고 SDK 형태로 배포하는 경우

### Dynamic Framework

리소스를 포함할 수 있고 소스 제공

| 장단점 | Static Framework | Dynamic Framework |
| ---- | ---------------- | ------------ |
| 장점  | 실행속도 빠르다         | 메모리 효율이 뛰어나며 최종 앱빌드시 컴파일 속도가 단축 |
| 단점  | 비교적 메모리 사용이 높고 최종 앱빌드시 컴파일 속도가 더 오래걸림 | 런타임시 실행되므로 런타임 속도에 영향을 줌 |

### Mach-O Type

* Executable - 앱 프로젝트 생성시 기본 설정.(실행가능한 바이너리)
* Dynamic Library - 동적 라이브러리.(프레임워크 프로젝트 생성시 기본 설정)
* Bundle - 리소스
* Static Library - 정적 라이브러리
* Relocatable Object File - Object Code로 구성된 파일.(사용해보지 않아서 잘 모르겠다)

### Xcode내에서 프레임워크 사용시 설정 옵션

복사해서 runtime 에 연결할 것인지 참조하여 build time 에 연결할 것인지 선택하는 옵션.

* Static Library - Do Not Embed 선택 (복사, build time)
* Dynamic Library - Embed & Sign, Embed Without Signing (참조, runtime)

## Xcode 아키텍쳐 정의

| 아키텍쳐 |    설명    |   기기 모델    |
| ------ | --------- | ------------ | 
| armv7  | arm cpu   | 아이폰4s 이전 기기 |
| armv7s | arm cpu   | 아이폰5, 아이폰5c |
| i386   | intel 32bit cpu (simulator) |  |
| x86_64 | intel 64bit cpu (simulator) | 2005년부터 2021년 사이에 출하된 Intel 기반의 Mac |
| arm64  | arm 64bit cpu | 아이폰5s 이후 기기 또는 2020년 말 이후에 출하된 Apple Silicon 기반의 Mac |

m1이전의 인텔맥들은 모두 cpu 아키텍쳐가 인텔이기때문에 시뮬레이터 아키텍쳐는 인텔 cpu였다.
x86_64 경우엔 amd아키텍쳐 기반이며 인텔아키텍쳐와 호환된다.

iOS디바이스들은 arm 아키텍쳐 기반으로 제작된 cpu를 사용하기 때문에 arm이라고 보면 된다.

## 인텔맥과 m1의 아키텍쳐

### 애플 실리콘

arm 기반으로 제작된 m시리즈로써 애플 자체 프로세서.

아이폰과 맥이 동일한 아키텍쳐를 사용하고 있으므로 맥과 아이폰에서 모두 실행가능한 상황이 도래했다.