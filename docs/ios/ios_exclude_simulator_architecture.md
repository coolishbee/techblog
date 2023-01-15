# 시뮬레이터 아키텍처용 arm64 제외시키기

iOS 에서 라이브러리 빌드후 cocoapods 에 올릴때 `pod lint` 시 의존도 라이브러리들 안에서 에러가 난다.

나의 경우에는 Firebase 의존성을 주입시켜서 발생했다.

이유는 iOS Simulator SDK 가 빌드옵션에 포함되어서 인데 이걸 podspec 파일에서 제외시키는 방법이 필요하다.



.podspec

```
s.pod_target_xcconfig = { 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64' }
s.user_target_xcconfig = { 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64' }
```



두줄만 추가해주면 해결된다.



> 출처
>
> https://stackoverflow.com/questions/63607158/xcode-12-building-for-ios-simulator-but-linking-in-object-file-built-for-ios

