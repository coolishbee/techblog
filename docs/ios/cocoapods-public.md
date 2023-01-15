## Cocoapod 으로 소스 공개(Public)용으로 프레임워크 배포

### PodSpec 파일 생성 및 편집하기

spec 파일만 따로 만드는 방법이 있다. 사실 이 파일만 있다면 배포가 가능하다.

```
$ pod spec create pod_name
```

.podspec 이 로컬에 생성 될 것이다.

```
Pod::Spec.new do |s|
  s.name                = "git-public-pod-public"
  s.version             = "0.1.5"
  s.summary             = "The SDK for iOS Swift provides a modern way of implementing APIs."

  s.description         = <<-DESC
                          Developed in Swift, the SDK for iOS Swift provides a modern way of implementing
                          LINE APIs. The features included in this SDK will help you develop an iOS app with
                          engaging and personalized user experience.
                          DESC

  s.homepage            = "https://github.com/james-chun-dev/git-public-pod-public"
  s.license             = "Apache License, Version 2.0"

  s.author              = "James"
  s.platform            = :ios, "13.0"
  
  s.module_name         = "SDK1"
  s.source              = { :git => "https://github.com/james-chun-dev/git-public-pod-public.git", :tag => "#{s.version}" }

  s.source_files        = 'SDK1/SDK1/SDK1.h'
  
end
```

더 많은 정보를 원한다면 cocoapods 공식사이트 문서 참고.

* name: pod spec 명령어로 만든 pod_name.
* platform: Podfile 상단에 있는 ios 버전 허용범위.
* source_files: sdk내에 포함될 소스 및 리소스. (번들)
* homepage: 공개된 페이지주소.
* author: 작성자이름. (특별히 중요하진 않아보임)
* module_name: framework 생성시 이름.
* source: git 저장소 주소.
* source_files: 공개할 소스파일.

만약 문제가 있더라도 편집된 .podspec 파일이 유효한지 체크해주는 명령어가 있다.

```
$ pod spec lint
$ pod spec lint --verbose specfilename.podspec //가급적이면 이렇게 쓰는게 좋다.(로그출력용)
```



### Git Repo 만들고 Tag 걸고 배포하기

위에 `pod spec lint` 에 성공하려면 git repo 주소가 존재해야하고 tag가 걸려있어야 한다.

git 사용법에 대해서는 생략하겠다.

![tag](https://user-images.githubusercontent.com/20632507/104142991-25c06b80-5401-11eb-826f-a95c7cfe81ef.png)

tag 버전과 sdk 버전이 일치해야 한다.( podspec 파일내 version)

### CocoaPods 에 배포하기

pod trunk 에 등록하는 세션요청 명령어. 재배포시에는 안해줘도 된다.

```
$ pod trunk register chc3484@gmail.com jamesChun
```

이렇게 이메일주소와 이름을 입력후 명령어를 실행하면 이메일주소로 확인메일이 발송된다.



![email](https://user-images.githubusercontent.com/20632507/104144508-cb2a0e00-5406-11eb-9a39-c981391a74c7.png)

링크를 클릭해주면 완료되고 다음 명령어로 업로드를 진행하면 된다.

```
$ pod trunk push pod_name.podspec
```

![success](https://user-images.githubusercontent.com/20632507/104155710-181fdb80-542b-11eb-9bfe-0b54e46ef576.png)

업로드 성공하면 이런 로그가 나온다. 

### Example Repository

* https://github.com/james-chun-dev/git-public-pod-public
* https://cocoapods.org/pods/git-public-pod-public

### 프레임워크 Cocoapods 재배포시

podspec 파일 유효성 검사

```
$ pod spec lint --verbose specfilename.podspec
```

trunk 에 등록된 세션정보 확인후 push.

```
$ pod trunk me
$ pod trunk push
```



### Podfile 사용법

```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '14.3'

workspace 'frameworkProj'
project 'frameworkTEST/frameworkTEST.xcodeproj'
project 'frameworkTESTAPP/frameworkTESTAPP.xcodeproj'

target 'frameworkTEST' do

	project 'frameworkTEST/frameworkTEST.xcodeproj'
	pod 'AFNetworking', '~> 3.0'
	pod 'PopupDialog', '~> 1.1'
end

target 'frameworkTESTAPP' do

	project 'frameworkTESTAPP/frameworkTESTAPP.xcodeproj'
	pod 'MBProgressHUD', '~> 1.2.0'
	pod 'git-public-pod-public', '~> 0.1'
end
```

공개용 프레임워크는 사용법이 워낙 잘 알려져 있기때문에 코멘트를 하지 않겠다.