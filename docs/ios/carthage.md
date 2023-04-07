
## 설치
home brew 를 이용해 설치합니다.
```
$ brew update
$ brew install carthage
```

## Cartfile 생성
.xcodeproj 또는 .xcworkspace가 있는 동일한 디렉토리에 Cartfile을 만듭니다.
```
$ touch Cartfile
```

## Cartfile 편집
Cartfile에 원하는 종속성을 나열합니다. 예를 들면 다음과 같습니다.
```
github "Alamofire/Alamofire" ~> 5.5
```

## 실행
```
$ carthage update --use-xcframeworks
```

## 빌드된 Framework 적용
Carthage/Build에서 빌드된 .xcframework 번들을 애플리케이션 Xcode 프로젝트의 "Frameworks and Libraries" 섹션으로 드래그합니다.

> Application에 Carthage를 사용하는 경우 "Embed & Sign"을 선택하고, 그렇지 않으면 "Do Not Embed"를 선택하십시오.