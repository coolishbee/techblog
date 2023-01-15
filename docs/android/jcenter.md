bintray 플러그인이 몇가지 있지만 com.novoda.bintray-release 을 사용하였다.



## Bintray 이용하기

1. [가입하기](https://bintray.com)
2. bintray Repository 생성(github repo가 아님)

![add new repo](https://user-images.githubusercontent.com/20632507/96668623-8e65f200-1396-11eb-8e26-42d474a669a9.png)

![create](https://user-images.githubusercontent.com/20632507/96668631-90c84c00-1396-11eb-80c7-76dd21fbdc16.png)

* Name : bintray Repository 이름
* Type : Maven 으로 설정하면 jcenter 도 사용가능
* Optional : 비어두고 Create 해도 됨

이렇게 레퍼지토리를 생성하면 배포할 준비는 끝난거다.



## github public 저장소 jcenter 로 배포하고 사용하기

우선 직접개발한 라이브러리를 jcenter에 배포하고 사용하려면 github 소스를 public 으로 설정해야 한다.

이유는 나중에 Add Jcenter 했을때 github 주소를 검수하는 듯하다. private 이면 404가 나기 때문에 

jcenter배포가 불가능하다.

##### build.gradle(project)

```java
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.0'
        classpath "com.novoda:bintray-release:0.9.2"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```

gradle 버전과 novoda 버전 호환성도 이슈가 있었다. (gradle 4.x.x 로 했더니 배포가 안되는 버그가 있었음)



##### build.gradle(sdk)

```
apply plugin: 'com.android.library'
apply plugin: 'com.novoda.bintray-release'


version = "0.1.17"

/** Configurations for com.novoda.bintray-release plugin.
 *
 * This plugin uses https://github.com/bintray/gradle-bintray-plugin internally.
 * See https://github.com/novoda/bintray-release/wiki/Configuration-of-the-publish-closure for more
 * detail about options in `publish` closure.
 */
publish {
    // Required.
    bintrayUser = 'james-chun-dev'
    bintrayKey = 'Edit Your Profile > API Key'
    groupId = 'com.universal.sdk'
    publishVersion = version
    artifactId = 'universalsdk'

    // Optional.
    userOrg = 'james-chun-dev'
    repoName = 'universal-sdk'
    desc = 'The Universal SDK for Android provides a modern way of implementing open APIs.'
    issueTracker = 'YOUR_ISSUE_TRACKER_URL'
    website = 'https://github.com/james-chun-dev/universal-sdk-android'
    repository = 'https://github.com/james-chun-dev/universal-sdk-android'
    dryRun = false
    override = true
}

/** Add support for syncing to maven central.
 *
 * This closure synchronizes the POM file that specifies transitive dependencies with the maven
 * central.
 * See https://github.com/novoda/bintray-release/wiki/Add-support-for-syncing-to-maven-central for
 * more detail.
 */
subprojects {
    group = this.group
    version = this.version

    def isUploading = project.getGradle().startParameter.taskNames.any {
        it.contains('bintrayUpload')
    }
    if (isUploading && project.name in [MODULE_NAME]) {
        apply plugin: 'maven'

        def defaultPomPath = "build/publications/maven/pom-default.xml"
        gradle.taskGraph.whenReady { taskGraph ->
            taskGraph.getAllTasks().find {
                it.path == ":$project.name:generatePomFileForMavenPublication"
            }.doLast {
                file(defaultPomPath).delete()

                // Overriding POM file to make sure we can sync to maven central.
                // See https://maven.apache.org/pom.html for more detail about POM.
                pom {
                    project {
                        name project.publish.artifactId
                        artifactId project.publish.artifactId
                        packaging project.name == 'compiler' ? 'jar' : 'aar'
                        url project.publish.website
                        version this.version
                    }
                }.writeTo(defaultPomPath)
            }
        }
    }
}
```

* bintrayUser : 콘솔 우측상단에 뜨는 유저이름

* bintrayKey : 콘솔 우측상단에 유저이름을 클릭하면 Edit Your Profile > API Key 가면 얻을 수 있다.

* groupId : 배포시 사용될 그룹 이름 (나는 라이브러리 패키지 주소와 맞췄다)

* publishVersion : 배포시 사용될 버전 이름

* artifactId : 배포시 사용될 sdk 이름

  ```
  <dependency>
  	<groupId>com.universal.sdk</groupId>
  	<artifactId>universalsdk</artifactId>
  	<version>0.1.16</version>
  	<type>pom</type>
  </dependency>
  ```

  3가지 키값들이 합쳐져서 아래와 같이 사용할 수 있게 된다.

  ```
  implementation 'com.universal.sdk:universalsdk:0.1.16'
  ```

* userOrg : bintrayUser 와 동일하게 적어주면 된다.

* repoName : 최초에 bintray 콘솔에서 생성한 레퍼지토리 이름

* desc : 라이브러리에 대한 설명

* issueTracker : 배포시에 크게 중요하진 않은거 같다.

* website : 깃허브 주소

* repository : 깃허브 주소 

* dryRun : 

* override : 

* subprojects : 

![bintrayUpload](https://user-images.githubusercontent.com/20632507/96670632-c4a57080-139a-11eb-82d1-54ee4841f54f.png)

bintrayUpload 하면 배포된다. bintray 콘솔창에 가보면 라이브러리가 업로드된 걸 확인할 수 있다.

### Jcenter 배포

라이브러리가 정상적으로 배포되면 Add to JCenter 라는 버튼이 활성화될 것이다.

Gihub 프로젝트가 public 이면 검수보내고 몇분뒤에 jcenter 에 배포가 된다.



## github private 저장소 maven 으로 사용하기

만약에 소스공개가 어려운 경우엔 이 방법을 쓸 수 있다.



#### maven 주소 복사하기

![url copy](https://user-images.githubusercontent.com/20632507/96678487-c4ae6c00-13ac-11eb-8075-3cde6075aef5.png)



#### maven url 사용하기

##### build.gradle(project)

```
buildscript {
    ext.kotlin_version = '1.3.11'
    repositories {
        google()
        jcenter()
        maven {
            url "https://dl.bintray.com/james-chun-dev/universal-sdk"
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven {
            url "https://dl.bintray.com/james-chun-dev/universal-sdk"
        }
    }
}

dependencies {
    implementation 'com.universal.sdk:universalsdk:0.1.16'
}
```



