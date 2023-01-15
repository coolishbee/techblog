갑작스레 잘 쓰고 있던 Jcenter 가 종료함에 따라 maven 을 처음 써보게 되었다.
물론 Jcenter 도 처음이었지만 굉장히 손쉽게 했었는데 maven 은 생각보다 쉽지 않아서 마이그레이션 가이드를 공유 해보려고 한다.

## 계정 생성

안드로이드 라이브러리를 Maven 에 배포하기 위해서는 Nexus repository manager 에 접근하려면 계정이 있어야 합니다.
Nexus repository manager 에서는 로그인만 제공되며 계정생성이 안되기 때문에 [sonatype](https://issues.sonatype.org) 을 통해 계정생성을 합니다.

![sign up](https://user-images.githubusercontent.com/20632507/117414676-63b64200-af52-11eb-8529-4eaf4523e4b3.png)

여기서 Username 과 Password 가 nexus repo 계정이 됩니다. 그리고 Username 은 계정생성후 변경이 되지 않더군요. (유의바람)

가입후 **Create** 버튼으로 이슈를 등록해야 한다. 아래 2가지 항목은 필히 선택해주고 나머지는 편집가능하다.
* Project : Community Support - Open Source Project Repository Hosting (OSSRH)
* Issue Type : New Project

>참고로 라이브러리 소스가 비공개여도 상관없습니다.

여기까지 성공하면 대쉬보드가 나올 것이고 2가지를 만족시켜주고 코멘트를 달아주면 됩니다.

![Dash board](https://user-images.githubusercontent.com/20632507/117416035-ee4b7100-af53-11eb-8592-417e861f188c.png)

* Group Id 를 io.github.* 형태로 변경
* 아래 코멘트중에 JIRA ticket 이라고 발급된 OSSRH-68408(필자경우)를 가지고 public git repo 생성
* 위 2가지가 만족됐다면 확인해달라고 코멘트 작성

자세한 진행 사항이 알고 싶다면 [나의 이슈](https://issues.sonatype.org/browse/OSSRH-68408) 를 참고바랍니다.
말하자면 여기까지 진행한 것들은 maven 에 업로드할 계정권한을 얻는 과정이라고 보면 될 것 같습니다.

## 저장소 배포 준비

코멘트에 관리자가 groupId has benn prepared 라고 남겼다면 [Nexus repository manager](https://s01.oss.sonatype.org) 로 로그인 가능해진겁니다. 위에서 만든 Username 과 Password 로 로그인합니다.

![Nexus repo Dashboard](https://user-images.githubusercontent.com/20632507/117418606-93ffdf80-af56-11eb-9503-6fcfe17cfa0c.png)

이런 페이지가 나올 것이고 Profile > User Token > Access User Token 를 누르면 토큰을 발급 받을 수 있는데 제 경우에는 토큰이 필요 없었습니다. 아마 pom.xml 을 직접 컨트롤하지 않고 gradle 스크립트를 이용해서 그런 것 같은데요. 일단 진행 해봅시다.

## 배포 스크립트 설정

자 이제 staging 이라는 저장소로 라이브러리를 스크립트로 통해 배포해 볼건데요.
스크립트 작성에 앞서 GPG 키를 만들어야 합니다.
* [Sonatype GPG 사용가이드](https://central.sonatype.org/publish/requirements/gpg)
* [GPG 한글가이드](https://github.com/jameschun7/document/blob/main/android/gpg-usage.md)

### GPG 키 인증
GPG 를 통해 발급받은 정보들을 프로젝트내 `gradle.properties` 파일에 설정해줍니다.
```
signing.keyId=YourKeyId
signing.password=your-jira-password
signing.secretKeyRingFile=../secring.gpg

ossrhUsername=your-jira-id
ossrhPassword=your-jira-password
```

### 배포
* 기존 jcenter 는 mavenCentral 로 변경
* `classpath "com.android.tools.build:gradle:3.4.0"` 를 사용함

`build.gradle` 스크립트 예제
```
apply plugin: 'com.android.library'
apply plugin: 'maven'
apply plugin: 'signing'

group = "io.github.jameschun7"
archivesBaseName = "universalsdk"
version = "0.2.3"

signing {
    sign configurations.archives
}

uploadArchives {
    repositories {
        mavenDeployer {
            beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

            repository(url: "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/") {
                authentication(userName: 'your-jira-id', password: 'your-jira-password')
            }

            snapshotRepository(url: "https://s01.oss.sonatype.org/content/repositories/snapshots/") {
                authentication(userName: 'your-jira-id', password: 'your-jira-password')
            }

            pom.project {
                name 'Universal Android SDK'
                packaging 'aar'
                // optionally artifactId can be defined here
                description 'The Universal SDK for Android provides a modern way of implementing open APIs.'
                url 'https://github.com/jameschun7/simple-firebase-sdk-unity'

                scm {
                    url 'https://github.com/jameschun7/universal-sdk-android.git'
                }

                licenses {
                    license {
                        name 'The Apache License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }

                developers {
                    developer {
                        id 'jameschun7'
                        name 'james chun'
                        email 'chc3484@gmail.com'
                    }
                }
            }
        }
    }
}
```

### 패키지명 변경
groupId 를 변경했다면 패키지명도 동일하게 맞춰줘야 한다.

[패키지명 변경하기](https://github.com/jameschun7/document/blob/main/android/android-package-refactor.md)

## maven 배포

```
gradle uploadArchives
```

### Staging 배포 확인

![](https://user-images.githubusercontent.com/20632507/117629562-41702e80-b1b5-11eb-8613-f112897204a2.png)

배포후 Staging Repo 에서 Refresh 버튼을 눌러주면 라이브러리가 올라간 것을 확인할 수 있다. 해당 레포지토리를 클릭하면 Close, Drop 버튼이 활성화 되며 Close 버튼을 눌러서 확인하면 검수절차가 진행된다.

![](https://user-images.githubusercontent.com/20632507/117629583-48973c80-b1b5-11eb-9933-6ab693b8f5c6.png)

검수 결과 실패 또는 성공에 대한 로그를 확인할 수 있고 성공시 Release 버튼이 활성화 된다.

![](https://user-images.githubusercontent.com/20632507/117629658-58168580-b1b5-11eb-9eaf-29aad8f7c7b0.png)

물론 릴리즈하면 라이브로 배포가 되는 것이다.

![](https://user-images.githubusercontent.com/20632507/117629675-5e0c6680-b1b5-11eb-921c-11f03b7f3564.png)

정상 배포가 완료되면 Staging Repo 의 상태는 Closed 가 된다.

### 라이브 배포 확인

약 2시간정도면 지라 대쉬보드쪽에 첫 배포에 대한 코멘트가 달릴 것이고 [maven 저장소](https://search.maven.org ) 에서 검색하면 배포한 라이브러리가 검색된다.

![comment](https://user-images.githubusercontent.com/20632507/117634385-cfe6af00-b1b9-11eb-9299-546d0d82a00e.png)


## 사용

* jcenter -> mavenCentral
* maven url 변경

```
buildscript {    
    repositories {
        google()
        mavenCentral()
        maven {
            url "https://repo1.maven.org/maven2/"
        }
    }
    ...
}

allprojects {
    repositories {
        google()
        mavenCentral()
        maven {
            url "https://repo1.maven.org/maven2/"
        }
    }
}

dependencies {
    implementation 'io.github.jameschun7:universalsdk:0.2.4'    
}
```