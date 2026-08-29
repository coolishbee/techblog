
### iOS Resolver 사용시 pod 설치오류

Ruby 버전 이슈

!!! failure annotate "iOS Resolver 에러 메시지"

    ERROR:  Error installing cocoapods:
    **There are no versions of cocoapods-downloader (>= 2.0) compatible with your Ruby & RubyGems. Maybe try installing an older version of the gem you're looking for?
    cocoapods-downloader requires Ruby version >= 2.7.4. The current ruby version is 2.6.10.210.**


Unity iOS Resolver는 내부적으로 bash 쉘을 사용합니다. 따라서 rbenv 등으로 Ruby 버전 설정을 zsh 쉘에만 적용한 경우, Resolver 실행 시 해당 설정을 인식하지 못하고 macOS에 내장된 시스템 Ruby가 동작하게 됩니다. 이로 인해 요구 버전보다 낮은 Ruby가 사용되어 CocoaPods 설치가 실패합니다.

