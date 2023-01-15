## gpg 설치

* [GPG 공식사이트 다운로드](https://www.gnupg.org/download/index.html)
* [gpg4win 프로그램 다운로드](https://www.gpg4win.org)

#### 공식 사이트 다운로드

![gpg download](https://user-images.githubusercontent.com/20632507/117421639-ae878800-af59-11eb-9db7-5068f93e282d.png)

![dmg download](https://user-images.githubusercontent.com/20632507/117421661-b47d6900-af59-11eb-9cb3-643eb636c706.png)

## gpg 키 생성

```
$ gpg --gen-key
```

![](https://user-images.githubusercontent.com/20632507/117424796-260ae680-af5d-11eb-8bb2-1a4aa0cb19e9.png)

![](https://user-images.githubusercontent.com/20632507/117424820-2c995e00-af5d-11eb-96f9-0656d271f68b.png)

키에 대한 비밀번호는 인증시에 사용된다.

## gpg secring.gpg 파일 생성

```
$ gpg --export-secret-keys -o secring.gpg
```

생성된 파일은 인증시에 사용된다.

## gpg keyid 얻기

```
$ gpg --list-keys --keyid-format short
```
![](https://user-images.githubusercontent.com/20632507/117623024-3e257480-b1ae-11eb-87bb-388edbc59c9f.png)

## 공개키 서버로 전송
send
```
$ gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys EE539F98
$ gpg2 --keyserver hkp://keyserver.ubuntu.com --send-keys EE539F98
```
result
```
gpg: sending key 07DAB54FDB18DBA0 to hkp://pool.sks-keyservers.net
gpg: sending key 07DAB54FDB18DBA0 to hkp://keyserver.ubuntu.com
```

[참고](https://stackoverflow.com/questions/27936119/gradle-uploadarchives-task-unable-to-read-secret-key/39573795#39573795)

[참고](https://stackoverflow.com/questions/19462617/no-public-key-key-with-id-xxxxx-was-not-able-to-be-located-oss-sonatype-org)