
* Identifiers 메뉴로 이동 후 "+" 버튼을 클릭해주세요.

![apple developer](https://user-images.githubusercontent.com/72954886/101319035-9de8be00-38a4-11eb-85cc-910c28f233c9.png)

* 애플 로그인(Sign In with Apple)을 진행한 유저의 정보를 전달받기 위한 Services ID를 등록하겠습니다.

![service-id-setting](https://user-images.githubusercontent.com/72954886/132818034-b52c479b-640e-4196-8ba6-2356afb7097a.png)

* Description : 애플 로그인시 게임명이 노출될 공간입니다. (변경이 가능합니다)
* Identifier : com.gamepub 도메인명이 포함되도록 작성해주세요. (단, AppID 와 다르게 작성해주세요.)

![register-service-id](https://user-images.githubusercontent.com/72954886/132818071-e5b0b0ab-cd21-4d3d-b715-a1171bad8360.png)

* Services ID가 등록되었다면 Identifiers 메뉴 리스트에서 등록한 Services ID를 클릭하여 설정 페이지로 이동해주세요.

![service id config](https://user-images.githubusercontent.com/72954886/101322397-4cdbc880-38aa-11eb-8107-2b73d95be58a.png)

* Primary App ID : 연결할 앱 ID 를 선택해주세요
* Domains and Subdomains : 시스템팀에 문의해주세요.
* Return URLs : 시스템팀에 문의해주세요.

![auth-config](https://user-images.githubusercontent.com/72954886/132829133-41eb9394-ef27-4f5a-b74c-356f63a49ff6.png)

* Save 후 Service ID 는 개발사에 전달해주세요.
* 아래 apple_client_id 값으로 Service ID 를 넣어주시면 됩니다.

##### launcherTemplate.gradle 설정

```groovy
dependencies {
    ...
    }

android {
    ...

    defaultConfig {
        ...
        
        resValue("string", "facebook_app_id", "com.your.app.id.here")
        resValue("string", "google_web_client_id", "com.your.client.id.here")
        resValue("string", "apple_client_id", "com.your.client.id.here")
    }
    ...
```