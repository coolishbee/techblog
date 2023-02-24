# Jenkins Setup

## Jenkins 서버구축

### macOS Installers for Jenkins LTS

#### homebrew installer

##### terminal command:

* Install the latest LTS version: brew install jenkins-lts
* Install a specific LTS version: brew install jenkins-lts@YOUR_VERSION
* Start the Jenkins service: brew services start jenkins-lts
* Restart the Jenkins service: brew services restart jenkins-lts
* Update the Jenkins version: brew upgrade jenkins-lts

한번 서비스를 시작하면 맥이 재시동되도 서비스를 다시 재시작하거나 시작할 필요가 없다.
기본 접속은 http://localhost:8080 이며 설정에서 변경가능하다.

[젠킨스 공식홈페이지](https://www.jenkins.io/download/lts/macos/)

## Blue Ocean 사용법

### 설치

Jenkins 관리 > 플러그인 관리 > 설치가능 > 필터 검색란에 blue 라고 쳐주고 설치하면 아주 간단하게 설치가 완료된다.

### 접속방법

Dashboard 에서 블루 오션 열기 를 클릭해도 되고 https://jenkins-server-url/blue 로 접속해도 된다.

## IP 접속 설정

Jenkins 관리 - 시스템 설정 - Jenkins Location - Jenkins URL 에서 IP Port 설정

### 파일 설정

해당 파일에서 IP Port 설정 편집

intel:
```
$ /usr/local/opt/jenkins-lts/2.xxx.x/homebrew.mxcl.jenkins-lts.plist
```

m1:
```
$ /opt/homebrew/Cellar/jenkins-lts/2.xxx.x/homebrew.mxcl.jenkins-lts.plist
```

> vi homebrew.mxcl.jenkins-lts.plist
```
<string>--httpListenAddress=127.0.0.1</string>
<string>--httpPort=8080</string>
```