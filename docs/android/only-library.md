## 앱 모듈을 라이브러리 모듈로 변환

앱없이 라이브러리 모듈만 포함한 프로젝트 생성방식.

1. **File > New > New Project**을 클릭합니다.

2. **No Activity Next > 대충 설정하고 Finish**

3. (Module: app) 의 `build.gradle` 파일을 엽니다.

4. `applicationId`를 지정하는 줄을 삭제합니다. Android 앱 모듈에서만 지정할 수 있습니다.

5. ```
   apply plugin: 'com.android.application'
   ```

   다음과 같이 변경합니다.

   ```groovy
   apply plugin: 'com.android.library'
   ```

6. 파일을 저장하고 **File > Sync Project with Gradle Files**를 클릭합니다.

7. app폴더안에 libs, src, build.gradle 파일들을 상위폴더로 이동합니다. 그리고 app폴더는 삭제합니다.

   ![app folder](https://user-images.githubusercontent.com/20632507/103523136-c7563300-4ebe-11eb-9ec6-283d02558427.png)

   ![folder delete](https://user-images.githubusercontent.com/20632507/103523294-fcfb1c00-4ebe-11eb-9ede-894bfd55cb9f.png)

8. settings.gradle 안에 내용을 지워줍니다. 파일을 삭제해도 상관없음

   ![settings](https://user-images.githubusercontent.com/20632507/103523316-04bac080-4ebf-11eb-9e48-eabedb1aa62d.png)

   ![empty](https://user-images.githubusercontent.com/20632507/103523341-0c7a6500-4ebf-11eb-9868-4402e79bada2.png)

9. build.gradle 스크립트 수정

   ```groovy
   apply plugin: 'com.android.library'
   apply plugin: 'kotlin-android'
   apply plugin: 'kotlin-android-extensions'
   
   //added 꼭 추가되야함!!
   buildscript {
       ext.kotlin_version = '1.3.11'
       repositories {
           google()
           jcenter()
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
       }
   }
   //end
   
   android {
       compileSdkVersion 30
       buildToolsVersion "30.0.1"
   
       defaultConfig {
           minSdkVersion 17
           targetSdkVersion 30
           versionCode 1
           versionName "1.0"
   
           testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
       }
   
       buildTypes {
           release {
               minifyEnabled false
               proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
           }
       }
   }
   
   dependencies {
       implementation  "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
   
   }
   ```

10. 여기까지 저장하고 **Sync Project with Gradle Files** 했을때 에러가 난다면 gradle-wrapper.properties 에 gradle 버전과 build gradle 버전이 맞는지 여부를 확인해봐야 합니다. 이것은 코틀린 버전과도 연관이 있을 수 있습니다.

11. 최종 프로젝트 뷰

    ![project view](https://user-images.githubusercontent.com/20632507/103524364-cc1be680-4ec0-11eb-8b6d-96fa6485dcb9.png)

12. 최종 프로젝트 파일뷰

    ![file view](https://user-images.githubusercontent.com/20632507/103524374-d1793100-4ec0-11eb-9d13-239d19357679.png)

