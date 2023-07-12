
### sed 문자열 편집 명령어

```
$ sed -i '' 's/"One"/"Google"/g' launcherTemplate.gradle
$ sed -i '' 's/"Google"/"One"/g' launcherTemplate.gradle
```

### 파일 검색 명령어

```
$ grep -n "android_store" launcherTemplate.gradle
$ cat redis.conf|grep '# requirepass'
$ find / -name redis.conf
$ find . -name "redis*"
```

### scp 파일 업로드 명령어

```
$ scp -i apkupload.key Google_UnityDemo.apk apkupload@127.0.0.1:/path/download
```

### fastlane 빌드 명령어

```
$ fastlane ios xcframework version:1.1.1
```

### jenkins

```
hudson.model.WorkspaceCleanupThread.disabled
```

### htpasswd

```
$ htpasswd -c .htpasswd id
```

### keytool 인증서 export 명령어

```
$ keytool -exportcert -alias universal -keystore universal.keystore | openssl sha1 -binary | openssl base64
```

### 유니티 터미널 빌드 명령어

```
$ sh '/Applications/Unity/Hub/Editor/2019.4.30f1/Unity.app/Contents/MacOS/Unity -projectPath "$WORKSPACE" -quit -batchmode -buildTarget Android -executeMethod BuildMenu.Buildmachine_TEST -logFile -'
```

```
$JENKINS_HOME/
//jenkins
-quit -batchmode -buildTarget iOS -executeMethod BuildMenu.Buildmachine_iOS
```