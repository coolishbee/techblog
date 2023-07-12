

### Docker 빌드 배포 실행 명령어

실행
```
$ sudo docker run --name test-api-server -p 80:80 -it -d efc4d894da83
```

m1(맥)에서 빌드
```
$ docker build --platform linux/amd64 -t marketing-report-go .
$ docker tag marketing-report-go test-repo.co.kr:443/marketing-report-go:0.5
$ docker push test-repo.co.kr:443/marketing-report-go:0.5
$ docker pull test-repo.co.kr:443/marketing-report-go:0.5

//기본 실행
$ docker attach $(docker run --name ads-report-go -p 7100:80 -it -d 0d767cb42376)
//백그라운드 실행 및 자동실행
$ docker attach $(docker run -d --restart always --name marketing-report-go -p 7000:80 -it -d e6414dc2dcbc)
```

기본 빌드
```
$ docker build -t pub-sdk-api .
$ docker tag pub-sdk-api test-repo.co.kr:443/pub-sdk-api:0.5
$ docker push test-repo.co.kr:443/pub-sdk-api:0.5
$ docker pull test-repo.co.kr:443/pub-sdk-api:0.5
$ docker attach $(docker run --name pub-sdk-api -p 5500:80 -it -d 12877777cdc5)
```

도커 로그
```
$ sudo docker logs --tail 20 -f 4323af3a4a39
```