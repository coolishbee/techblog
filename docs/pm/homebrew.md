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

## [mysql] v5.7 설치 및 초기 설정

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