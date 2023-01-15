# acra 사용법

런타임 방식으로써 크래시발생시 acra http API 를 사용하는 방법 가이드.

## 테스트 환경

* com.android.tools.build:gradle:4.0.1
* ACRA Version 5.7.0

## 제공되는 기능

* Http: 크래시리포트를 rest api 로 송신.
* Email: 이메일주소로 크래시리포트 발송.
* Dialog: 크래시 발생시 다이얼로그 이벤트.
* Notification: 크래시 발생시 푸쉬알람.
* Toast: 크래시 발생시 토스트 메시지.

## 설정

* 컴파일 방식: annotation 을 이용하여 미리 설정정보를 셋팅하는 방식이다.(AndroidManifest.xml 필수)
* 런타임 방식: 모든 설정정보는 API 소스코드로 작성하는 방식이다.

## 예제

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.d("MainActivity", "onCreate");

        CoreConfigurationBuilder builder = new CoreConfigurationBuilder(this);
        builder.setBuildConfigClass(BuildConfig.class).setReportFormat(StringFormat.JSON);
        builder.getPluginConfigurationBuilder(HttpSenderConfigurationBuilder.class)
                .setUri("http://192.168.12.10:55000/send")
                .setHttpMethod(HttpSender.Method.POST)
                .setBasicAuthLogin("james") //backend htpasswd 설정필요
                .setBasicAuthPassword("1234")
                .setEnabled(true);

        ACRA.DEV_LOGGING = true;
        ACRA.init(getApplication(), builder);

        throw new NullPointerException();
    }
}
```

## 문서

* [ACRA/acra](https://github.com/ACRA/acra)
* [API Reference](https://github.com/ACRA/acra/wiki)
* [Examples](https://github.com/ACRA/acra/wiki/Examples)

# acra-go 사용법

안드로이드 ACRA 을 위한 백엔드 서버.

## 설치

```
go install -v github.com/gen2brain/acra-go/acra-go
```

터미널에서 명령어를 진행하면 `$GOPATH/bin/acra-go` 에 설치가 됩니다.
(acra-go 가 사용하고 있는 써드파티 의존성 해결 필요.)

* [go 개발환경 설정](https://github.com/jameschun7/docs/blob/main/golang/golang-develop-env.md)
* [go mod 사용법](https://github.com/jameschun7/docs/blob/main/golang/golang-develop-env.md)

go mod 사용법을 참고하여 프로젝트내에서 `go mod init` 명령어를 실행하면 vendor 파일 읽고 의존성 걸려 있는 모듈들을 모두 받는듯 하다. (패키지매니저 역할)

## 설정

### Server

```
acra-go -bind-addr 192.168.12.10:55000 --htpasswd-frontend .htpasswd
```

### Client

* [예제 참고](#예제)

### htpasswd 파일 생성

```
htpasswd -c .htpasswd id
```

## Usage

명령어 정리

* -bind-addr string : Bind address (default ":55000")
* -database-dir string : Path to database directory (default ".")
* -htpasswd-backend string : Path to htpasswd file, if empty backend auth is disabled
* -htpasswd-frontend string : 이 설정을 하면 웹프론트 페이지에 아이디 비번을 걸어서 로그인화면을 띄울 수 있다. htpasswd 파일을 만들고 file path만 지정해주면 된다. 사용하지 않으면 자동 비활성화된다.
* -read-timeout int : Read timeout (seconds) (default 5)
* -write-timeout int : Write timeout (seconds) (default 15)

## Pages

* `/`: dashboard of the crashes
* `/send`: accepts POST and PUT requests from you app and stores report in db
* `/view`: view all reports, returns json
* `/view?id=9823648d-20f6-4663-b8b5-f66f9fc97f81`: view a single report, identified by report id

## Screenshot

![Login](https://user-images.githubusercontent.com/20632507/115683507-7bf26280-a391-11eb-96c1-160db3df183d.gif)

![Front View](https://user-images.githubusercontent.com/20632507/115683529-844a9d80-a391-11eb-94b5-0fa76288eccf.gif)

## 참고

* [gen2brain/acra-go](https://github.com/gen2brain/acra-go)