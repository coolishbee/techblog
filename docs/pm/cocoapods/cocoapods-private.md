## Cocoapod 으로 소스 비공개(private)용으로 프레임워크 배포

### Git repo 만들기

git 주소가 2개가 필요하다.

* git lib repo (라이브러리 프로젝트)
* git specs repo (public 에서 사용한 cocoapod repo 역할을 할 git spec파일 저장소)

우선 라이브러리 git repo 가 있다는 전제하에 설명한다. 물론 podspec 파일도 셋팅되어있어야 한다.

.podspec 파일을 편집해주고 파일 유효성 검사. 이건 공개용과 동일하다.

```
$ pod spec lint --verbose specfilename.podspec
```

빈 레퍼지토리 https://github.com/james-chun-dev/specs.git 하나를 private 으로 만들었다.

그리고 pod repo 에 add 해준다.

```
$ pod repo add git-private-pod-private https://github.com/james-chun-dev/specs.git
```

![repo dir](https://user-images.githubusercontent.com/20632507/104162444-f6792100-5437-11eb-8e9d-20f152268d67.png)

정상적으로 완료되면 위와 같이 /Users/.cocoapods/repos 에 폴더가 생성된다.

(README.md 는 git repo 만들때 임의로 생성한 파일)

push 명령어를 실행한다.

```
$ pod repo push git-private-pod-private git-private-pod-private.podspec
```

![log](https://user-images.githubusercontent.com/20632507/104159737-3689d500-5433-11eb-9355-18d443a0e52e.png)

그랬더니 에러가 나네...음 tag 를 안걸어서 나는 에러였다. git 라이브러리 프로젝트 repo 에다 tag 를 걸어주고 다시 했더니

![push](https://user-images.githubusercontent.com/20632507/104159753-3be71f80-5433-11eb-90f6-3919c6f9be70.png)

성공!

아까 등록한 https://github.com/james-chun-dev/specs.git 에 가보니 podspec 폴더에 태그가 걸려서 올라와있다.

![specs](https://user-images.githubusercontent.com/20632507/104159774-45708780-5433-11eb-90f3-cb9efa266ca5.png)

이렇게 되면 성공한거다. 

### 프레임워크 재배포시

podspec 파일 유효성 검사

```
$ pod spec lint --verbose specfilename.podspec
```

pod repo 에 push

```
$ pod repo push [폴더명] specfilename.podspec
```



### 팀 동료가 접근시

다른 mac 에서 사용하려면 처음 만든 specs.git 저장소를 이용해서 로컬로 내려받아야 한다.

```
$ pod repo add [폴더명] https://github.com/jameschun7/specs.git
```



### Podfile 사용법

pod file 에서의 사용법은 약간 다른데 어느정도 예상된 방법이다.

이 내용은 찾아보면 생각보다 많이 나온다ㅎㅎ

```
pod 'git-private-pod-private', :git => 'https://github.com/james-chun-dev/git-private-pod-private'
```

이렇게 해도 되고 pod file 상단에 spec git repo 를 넣어주고

```
source 'https://github.com/james-chun-dev/specs.git'

pod 'git-private-pod-private'
```

이렇게 해도 된다. 아마도 여러 private 라이브러리용 spec git 으로 활용해도 괜찮을듯 싶다. 새로 spec 용 git 을 파지않고!!

### Podfile 전체내용

```
source 'https://github.com/james-chun-dev/specs.git'
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
	pod 'git-private-pod-private'
end
```

![test](https://user-images.githubusercontent.com/20632507/104257054-e0af3e80-54bf-11eb-8ffb-6c214d7448e8.png)

공개용과 비공개용 pod 테스트를 동시에 해보았더니 문제없이 받아진다.

### Example Repository

* https://github.com/jameschun7/cocoapod-google-firebase-dependency-solution.git