
[Docker Compose 커맨드 사용법](https://www.daleseo.com/docker-compose/)

## docker-compose.yaml 파일 구성

```
# 도커 컴포즈의 스키마 버전입니다. 이 스키마 버전은 docker의 버전에 따라 지원되는 버전이 달라집니다.
# 되도록 최근의 버전을 사용하는 것이 좋습니다.
version: "3"
 
# 애플리케이션의 일부로 실행하려는 서비스 목록을 정의 합니다.
# 서비스 이름은 임의로 선택 할 수 있습니다.
# 저는 이전글의 도커 파일로 생성했던 이미지를 한번 사용해 보았습니다.
services:
  app1:
    #build를 사용하게 된다면 image 항목을 사용하지 않아도 도커 컴포즈가 실행됩니다.
    #build :
      # 빌드 명령을 실행할 디렉터리 경로
      #context: .
      # 도커 이미지를 빌드하는데 사용할 도커 파일 위치
      #dockerfile: ./Dockerfile

    # 이미지 셋팅입니다
    image: demo:latest

    # 커맨드의 변경이 필요하다면 여기서 재 정의를 할 수 있습니다
    #command: /bin/bash -c "java -jar demo-0.0.1-SHAPSHOT.jar"

    # 노출시킬 포트 입니다.
    # - 로 여러개의 포트를 노출 시킬 수 있습니다
    ports:
    - 8080:8080
    # 작업 디렉토리를 지정해 줄 수 있습니다.
    # working_dir: /app

    # 마운트 할 볼륨입니다. 이 부분은 docker로 생성할 때 지정했던 부분과 거의 일치 합니다.
    # 상대경로로 입력도 가능합니다. ex) ./:/app
    volumes:
     - C:/Users/admin/Desktop/volume/log:/volume
    depends_on:
    # 의존 관계 설정
    - database
   
    # 데이터 베이스가 필요하거나 다른 컨테이너와의 통신이 필요하다면, 이 항목을 통해 연결할 수 있습니다.
    # 단, 파일내 정의된 다른 서비스여야 연결이 가능합니다.
    # 만약 버전을 도커 컴포즈 3 버전 이상을 사용했다면 docker-compose.yml 안에 있는 서비스들은 별도로 지정하지 않으면 하나의 네트워크에 속합니다.

    # links:
    #  - database
 
    # 네트워크 모드를 설정할 수 있습니다. 기본적으로는 도커안의 내부 네트워크를 사용하게 됩니다.
    # network_mode: host
 
  database:
    # 'database'서비스에서 참조할 이미지
    image: mariadb:latest
    ports:
    - 3306:3306
    # 만약 컨테이너가 예상치 못한 일로 kill 되어도 바로 다시 띄울 수 있는 옵션 입니다.
    restart: always
    environment:
        # 환경 설정에 필요한 설정들 입니다.
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE : database
      MYSQL_USER: root
      MYSQL_PASSWORD: 1234
```

## 커맨드 명령어

```
# 도커 컴포즈 컨테이너들을 백그라운드로 띄우기
$ docker-compose up -d
 
# 도커 컴포즈 컨테이너들을 포어그라운드로 띄우기
$ docker-compose up
 
# 도커 컴포즈 컨테이너들을 내리기
$ docker-compose down
 
# 도커 컴포즈 컨테이너들을 다시 시작하기
$ docker-compose restart
 
# 도커 컴포즈 컨테이너들의 로그를 계속해서 읽기
$ docker-compose logs -f
 
# 도커 컴포즈 컨테이너들의 상태 확인
$ docker-compose ps
 
# 도커 컴포즈 설정을 확인
# 주로 -f 옵션으로 여러 개의 설정 파일을 사용할 때, 최종적으로 어떻게 설정이 적용되는지 확인해볼 때 유용합니다.
$ docker-compose config
 
# 다른 경로에 있는 도커 컴포즈 파일 사용
# 도커 컴포즈로 다른 이름이나 경로의 파일을 Docker Compose 설정 파일로 사용하고 싶다면 -f 옵션으로 명시를 해줍니다.
$ docker-compose -f /app/docker-compose.yml up
# 여러개의 도커 컴포즈 설정 파일을 사용할 수 있습니다. 이 때는 나중에 나오는 설정이 앞에 나오는 설정보다 우선하게 됩니다.
$ docker-compose -f docker-compose.yml -f docker-compose-test.yml up
```