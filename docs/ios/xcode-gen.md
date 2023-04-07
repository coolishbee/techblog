
## brew 로 설치:
```
brew install xcodegen
```

## project.yml 파일 편집:
```
name: ProjectName
targets:
  ProjectName:
    type: application
    platform: iOS
    deploymentTarget: "15.0"
    sources: [ProjectName]
    settings:
      base:
        INFOPLIST_FILE: ProjectName/Info.plist
        PRODUCT_BUNDLE_IDENTIFIER: com.example.ProjectName

```

## 터미널 명령어 실행:
```
xcodegen generate
```

>출처:
https://github.com/yonaskolb/XcodeGen