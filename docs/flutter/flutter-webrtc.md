# flutter-webrtc-demo 샘플 예제 따라하기

## 맥환경에서 flutter-webrtc-demo 클라이언트 따라하기

!!! github주소

    [https://github.com/cloudwebrtc/flutter-webrtc-demo](https://github.com/cloudwebrtc/flutter-webrtc-demo)

우선 깃허브 Usage 순서대로 따라하면 된다.

-   `git clone https://github.com/cloudwebrtc/flutter-webrtc-demo`
-   `cd flutter-webrtc-demo`
-   `flutter packages get`
-   `flutter run`

단, 디바이스가 연결되어있어야 동작한다. 이미 완성되어있는 프로젝트를 flutter 명령어만으로 빌드해서

디바이스에 인스톨후 실행시켜준다.

#### 워크스페이스 프로젝트를 열어서 디바이스에서 실행시키기

![device run](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FK4t2C%2FbtqFXg2t6fE%2FveO5gkt2QC6rhsLS6TPqnk%2Fimg.png)

그냥 디바이스 연결하고 Run 해주면 된다.

## flutter-webrtc-server 서버 따라하기

webrtc 로 디바이스간 화상채팅서비스를 제공하려면 서버가 필요하다.

그것은 디바이스간에 p2p 연결을 해줄 서버라고 생각하면 된다.(시그널링, 턴, 스턴, 아이스 등)

!!! github주소

    [https://github.com/flutter-webrtc/flutter-webrtc-server](https://github.com/flutter-webrtc/flutter-webrtc-server) 소스 내려 받고 서버 구축하기.

가이드대로 다운로드후 실행해준다.

Run하게 되면 WebRTC 서버가 올라가게 되고 자동으로 활성화 될 것이다.

이때 클라이언트에서 p2p call sample 인풋창에 아무것도 붙이지 않고 로컬IP 만 넣어주면 된다.

```
192.168.121.24
```

이런식으로 넣어주면 알아서 포트번호 8086이 붙어서 방이 만들어지고

목록에 디바이스명과 아이디로 방이 생성된다.

그다음 연결하고 싶은 디바이스 하나를 더 실행시켜서 동일한 아이피 명을 넣어주면

서로 연결할 수 있게 된다.

![log view](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvPqiR%2FbtqFSZ9DLAp%2FLMh8iDSsG1f8NkFkuoDh2k%2Fimg.png)

!!! success

    연결된 후 webrtc 서버 로그화면