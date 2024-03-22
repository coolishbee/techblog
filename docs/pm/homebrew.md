---
comments: true
---

## brew 설치

[공식사이트](https://brew.sh/index_ko) 를 통해 설치합니다.(m1 이든 intel 이든 설치명령어는 같습니다.)

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 환경변수

m1 은 설치된 바이너리 디렉토리가 달라서 환경변수 설정을 해줘야 합니다. (intel은 자동설정됨)

```
export HOME_BREW="/opt/homebrew/bin"
export PATH=$PATH:$HOME_BREW
```

## mysql 설치 및 초기 설정

brew 명령어로 설치:
```
$ brew install mysql@5.7
```

서비스 시작:
```
$ brew services start mysql@5.7
```

## mysql 폴더 삭제

인텔맥:
```
$ rm -rf /usr/local/var/mysql
```
m1:
```
$ rm -rf /opt/homebrew/var/mysql
```

## 명령어 정리

- `brew info packageName` : 패키지 정보 보기
- `brew outdated` : 업데이트 가능한 패키지 나열
- `brew upgrade packgeName` : 해당 패키지 업데이트
- `brew upgrade` : 업데이트 가능한 모든 패키지 업데이트
- `brew cleanup packgeName` : 해당 패키지의 최신버전을 제외한 나머지 버전들 모두 삭제
- `brew uninstall packgeName` : 해당 패키지 언인스톨

## 문제 해결

에러메시지
```
fatal: couldn't find remote ref refs/heads/master
```

다트 패키지 디렉토리정보가 변경된 듯 하다...
```
brew tap --repair
brew cleanup
brew update-reset
```

!!! 출처
    [https://stackoverflow.com/questions/75509911/fatal-couldnt-find-remote-ref-refs-heads-master](https://stackoverflow.com/questions/75509911/fatal-couldnt-find-remote-ref-refs-heads-master)