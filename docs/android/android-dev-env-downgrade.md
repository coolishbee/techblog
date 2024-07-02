---
comments: true
---

# 사용자를 위한 Android 개발환경 다운그레이드

## 히스토리
인생을 살면서 정말 이렇게 단 기간에 회사를 나온 경우는 없었는데 그럼에도 최선을 다했기에 한 달 동안 값진 경험에 대해 트러블슈팅을 정리해 보았다.

보통 고도화 목적으로 Groovy DSL 에서 Kotlin DSL 로 마이그레이션을 진행하지만 Android SDK 경우에는 낮은 사용자 환경을 고려해야 한다.<br>
그러나 전임자는 그런 것들을 고려하지 않은 채 jdk17, gradle 8.x, kotlin 1.9.x 환경에서 Kotlin DSL 로 마이그레이션 해버렸다.
그로인해 사용자 환경이 jdk11 이상 kotlin 1.9.x 가 아니면 SDK 모듈을 적용할 수 없게 되버린 상태였다.

물론 kotlin dsl 이란 굉장한 장점들이 있지만 jdk나 gradle, kotlin 의 꽤 높은 최신버전들에서만 제공되는 gradle API들 대부분이다.
특히나 모듈을 개발하는 입장이라면 가급적 groovy를 유지해야 트러블슈팅을 줄일 수 가 있다고 생각한다.

## Kotlin DSL 이란

여러가지 장점들이 많지만 가장 핵심적인 것은 어려운 그루비 문법이 아닌 간결하고 가독성이 뛰어난 코틀린을 gradle 환경구성에서 사용할 수 있는 점이다.

[그루비에서 코틀린](https://www.codeconvert.ai/groovy-to-kotlin-converter), [코틀린에서 그루비](https://www.codeconvert.ai/kotlin-to-groovy-converter) 문법적으로 변환해주는 툴들이 존재하기 때문에 문법적인 변환은 그리 어렵지가 않다.

그리고 [공식문서](https://developer.android.com/build/migrate-to-kotlin-dsl?hl=ko)나 구글검색을 통하면 [좋은 자료](https://blog.jetbrains.com/ko/kotlin/2023/05/kotlin-dsl-is-the-default-for-new-gradle-builds/)들이 많고 간단하게 장점을 적으면 이렇다.

* 코틀린 사용가능
* 한 프로젝트내에 멀티모듈 배포에 대한 재사용성
* IDE 단에서 빌드컴파일 전에 문법적인 에러 검출

## 마이그레이션 과정

현재 전임자가 만들어둔 빌드배포용 코틀린 스크립트를 살리기 위해 Kotlin DSL 를 유지하면서 jdk버전과 gradle 환경을 낮추는 작업을 진행했다.
단순히 버전만 내린다고 해결되는 문제가 아니다. 전체적인 gradle 포맷과 gradle api가 완전히 다르기 때문이다.

기준은 각각 서로 호환가능한 jdk8, gradle 6.8.3, gradle plugin 4.2.2, kotlin 1.5.10 으로 잡고 진행하였다.

project build.gradle.kts
```
plugins {
    id("com.android.application") version "8.2.2" apply false
    id("com.android.library") version "8.2.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.22" apply false
    kotlin("plugin.serialization") version "1.9.22" apply false
}
```
to
```
buildscript {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://s01.oss.sonatype.org/content/repositories/snapshots/")
        }
    }
    dependencies {
        classpath("com.android.tools.build:gradle:4.2.2")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.5.10")        
    }
}
allprojects {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://s01.oss.sonatype.org/content/repositories/snapshots/")
        }
    }
}
tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

setting.gradle.kts
```
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url = URI("https://s01.oss.sonatype.org/content/repositories/snapshots/")
        }
    }
}

```
to
```
rootProject.name = "Android-Project"
include(":lib:module1")
include(":lib:module2")
include(":lib:module3")
```

app 또는 library build.gradle.kts
```
import io.imqa.android.Config
import io.imqa.android.Versions

plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.android")
    kotlin("plugin.serialization")
    `maven-publish`
    signing
}

android {
    namespace = "packageName"
    compileSdk = Config.TARGET_SDK

    defaultConfig {
        minSdk = Config.MIN_SDK
    }

    buildTypes {
        all {
            buildConfigField(
                ...
            )            
        }

        release {
            isMinifyEnabled = false
        }
    }

    publishing {
        singleVariant("release")
    }

    buildFeatures {
        buildConfig = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    ...    
    implementation("org.jetbrains.kotlin:kotlin-stdlib:1.9.22")
    implementation(project(":lib:module1"))    
}

ImqaMavenPublisher.configure(
    project = project,
    groupId = "",
    artifactId = "",
    version = Versions.LIBRARY_VERSION,
)
```
to
```
import io.imqa.android.Config
import io.imqa.android.Versions

plugins {
    id("com.android.library")
    id("kotlin-android")
    id("maven-publish")
    id("signing")
}

android {    
    compileSdkVersion(Config.TARGET_SDK)

    defaultConfig {        
        minSdkVersion(Config.MIN_SDK)
        targetSdkVersion(Config.TARGET_SDK)
    }

    buildTypes {
        all {
            buildConfigField(
                ...
            )
        }

        getByName("release") {
            isMinifyEnabled = false
        }
    }

    buildFeatures {
        buildConfig = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    ...
    implementation("org.jetbrains.kotlin:kotlin-stdlib:${Versions.KOTLIN_VERSION}")
    implementation(project(":lib:module1"))
}

MavenPublisher.configure(
    project = project,
    groupId = "",
    artifactId = "",
    version = Versions.LIBRARY_VERSION
)
```

위와 같은 방법으로 수정하여 jdk나 kotlin 버전이 낮은 사용자 환경에서도 잘 작동되는 것을 확인하였다.