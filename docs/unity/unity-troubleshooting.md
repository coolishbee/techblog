
### iOS Resolver 사용시 pod 설치오류

ruby 버전 이슈(인텔맥).
unity ios resolver 내부적으로 bash 쉘을 사용하므로 ruby 버전설정을 zsh 쉘으로 설정되있는 경우
정상적으로 ruby path 를 못 찾고 내장 ruby가 작동되는 원인.

https://blog.yozi.kr/entry/mac에서-ruby-재설정설치-하기-rbenv
https://github.com/googlesamples/unity-jar-resolver/issues/654


!!! failure annotate "iOS Resolver"

    ERROR:  Error installing cocoapods:
    **There are no versions of cocoapods-downloader (>= 2.0) compatible with your Ruby & RubyGems. Maybe try installing an older version of the gem you're looking for?
    cocoapods-downloader requires Ruby version >= 2.7.4. The current ruby version is 2.6.10.210.**

### UnityHub Lincense 실패오류

UnityHub 를 통해 로그아웃후 재로그인하면 해결된다.

!!! warning "Unity Hub"

    Activation of your license failed. Try again or contact Unity Support for help


<!-- warning
abstract
info
success
danger
question
failure
bug
example
quote
note -->