## Cocoapod 설치

macOS 에는 Ruby 가 기본 내장되어 있다. 다음 명령어면 설치가 완료된다.

```
$ sudo gem install cocoapods
```

## Cocoapod 명령어

### pod init

새로운 pod 파일을 생성해준다.

```
$ pod init
```

### pod install

pod을 프로젝트에 세팅하기 위하여 맨 처음에 사용됩니다. 하지만 Podfile의 pod을 **추가, 수정, 삭제**할 때에도 사용됩니다.

`pod install` 명령어를 실행하면 **새로운 pod을 다운받고 설치**합니다. 그리고 각 pod 마다 설치된 버전을 **Podfile.lock** 에 기록해 놓습니다. Podfile.lock은 설치된 pod들의 버전을 계속 추적하여 기록해놓고 유지시키는 역할을 합니다.

`pod install` 을 실행하면,

Podfile.lock에 리스트된 팟들에 대해선 지정된 버전만 다운받습니다. 새로운 버전이 존재하는지 체크하지 않는 것이죠!

Podfile.lock에 리스트되지 않은 팟들은 Podfile에 명시된 버전 조건으로 검색하여 다운로드 받습니다. (ex. pod 'MyPod', '~>1.2')

```
$ pod install
```

### pod update

`pod update {팟이름}` 을 실행시키면, 코코아팟은 해당 팟의 업데이트된 버전이 있는지 검색합니다. 

Podfile.lock을 참조하지 않죠. 이 명령어는 팟을 **최신 버전으로 업데이트** 시켜주는 것입니다. (단, Podfile의 버전 조건과 일치해야 합니다.) 

단순하게 `pod update` 만 실행시키면 코코아팟은 모든 팟에 대해 가능한 최신 버전으로 업데이트를 실행합니다.

```
$ pod update
```

### pod outdated

pod outdated 를 실행하면, 코코아팟은 Podfile.lock에 리스트된 것보다 새로운 버전을 가진 모든 팟을 나열합니다. 

이 팟들에 대해 pod update {팟이름} 을 실행한다면 업데이트가 될 것이라는 것을 의미합니다. (역시나 Podfile의 버전 조건과 부합하는 한!)

### pod repo update

`/Users/{사용자이름}/.cocoapods/repos` 에 있는 모든 podspec 파일을 업데이트 합니다. podspec 파일에는 해당 pod 의 주소 등 중요한 정보들이 담겨있습니다.

``` 
spec.source = { :git => 'https://github.com/Alamofire/Alamofire.git', :tag => 'v3.1.1' }
```

`~/.cocoapods/repos` 에는 모든 pod에 대해 가능한 버전들의 `podspec` 파일들이 모여있습니다. `pod repo update` 를 실행하게 되면 최신 `podspec` 파일들로 업데이트되게 되는 것입니다. 추가한 라이브러리에 대한 `podspec` 이 업데이트되지 않아 오류가 날 경우 이 명령어를 통하여 해결할 수 있습니다.

### pod repo list

pod repo 리스트를 출력해줍니다.

```
$ pod repo list
```

### pod repo remove

특정 repo(private) 를 삭제합니다.

```
$ pod repo remove sdk_name
```

### pod spec lint

podspec 정보에 대한 유효성을 검사합니다. (xcode 빌드가 이루어짐)

```
$ pod spec lint --verbose UniversalSDK.podspec
```

### pod repo push

**pod spec lint** 에 성공했을 경우에만 사용가능하다.

로컬에 pod repo 를 push 해주고 동시에 private spec git 저장소에도 push 한다.

```
$ pod repo push [폴더명] UniversalSDK.podspec
```

### pod repo add

private repo 를 팀 동료에게 제공시 즉, 다른 mac 에서 private repo 를 접근시 사용해야 한다.

private spec repo 를 git 을 통해 내려받는다.

```
$ pod repo add [폴더명] https://github.com/jameschun7/specs.git
```

### cocoapods 모든 파일 삭제

```
$ rm -rf ~/.cocoapods
```

### 삭제후 초기화

```
$ pod setup
```

## Podfiles 파일 설정

Podfiles는 CocoaPods의 설정 파일이다. `pod init`을 통해서 파일을 생성할 수 있다.

```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'

target 'SampleSDK' do
    project 'SampleSDK/SampleSDK.xcodeproj'
    pod 'AFNetworking', '~> 3.0'
end
```

* pod: 사용하는 오픈소스 라이브러리 이름. 특정 버전, 브랜치, 경로, 저장소, 특정 커밋, 특정spec URL 등으로 원하는 시점의 소스 코드를 정의할 수 있다.
* target: 프로젝트의 특정 타겟에서만 반영되는 추가 설정. 주로 테스트 타겟의 경우에 테스트용 라이브러리 설정에 사용된다.
* xcodeproj: 프로젝트 이름과 워크스페이스 이름이 다른 경우 설정 옵션.

