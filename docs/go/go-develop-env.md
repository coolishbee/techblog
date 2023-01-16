## golang 설치

[공식사이트](https://golang.org/doc/install) 를 통해 설치합니다.

## 환경변수

윈도우든 맥이든 환경변수를 설정해야 합니다.

* GOROOT : /usr/local/go (설치시 자동 설정됨)
* GOPATH : workspace 에 해당하므로 임의로 정하면 됩니다.
* GOBIN : $GOPATH/bin

특히 GOROOT, GOPATH 가 제대로 설정되있어야 mod 명령어가 잘 작동됩니다.

환경변수 설정은 각자 알아서 본인취향대로 하면 됩니다.

## mod 사용법

* go mod init

  기본적으로 mod 파일을 생성해주고 vendor 폴더가 존재한다면 vendor.json 파일을 읽고 mod 파일을 생성하여 내부 프로젝트에서 사용되고 있는 의존성 소스에 대해 내려받고 관리해준다.  

* go mod vendor

  vendor 디렉토리를 생성해줍니다

* go mod tidy

  프로젝트 내부 의존성을 토대로 mod 파일 업데이트


## 참고

* [Go Modules 살펴보기](https://velog.io/@kimmachinegun/Go-Go-Modules-살펴보기-7cjn4soifk#go111module)

