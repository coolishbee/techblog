일단 fastlane 을 설치하는 방법은 여러가지가 있다.

- bundler
- brew
- macOS 의 system ruby

------

## Xcode command line tools (macOS)

clt 는 git, svn, perl, python 등 여러 컴파일러를 제공해줍니다. (보통 homebrew 설치시 자동설치됩니다)

뭘로 하든 기본설치라는 말씀.

```no-highlight
xcode-select --install
```

## Installing *fastlane*

*fastlane*은 여러 가지 방법으로 설치할 수 있습니다. 선호하는 방법은 [*Bundler*](https://bundler.io/)입니다. *fastlane*은 Homebrew를 통해 직접 설치할 수도 있습니다(macOS의 경우). macOS의 시스템 Ruby를 사용하는 것은 가능하지만 종속성을 관리하기 어렵고 충돌을 일으킬 수 있으므로 권장하지 않습니다.

### Managed Ruby environment + Bundler (macOS/Linux/Windows)

ruby는 시스템루비가 아닌 버전(2.5 이상)을 새로 설치하고 bundler 를 이용해서 fastlane 을 설치하는 가이드입니다.(bundler를 설치하려면 gem 이라는 ruby 라이브러리 패키지가 필요하다. 선행조건)

#### Ruby

macOS를 사용하는 경우 시스템 Ruby는 권장하지 않습니다. [시스템 환경을 변경하지 않고 Ruby를 설치하는 방법은 다양합니다](https://www.ruby-lang.org/en/documentation/installation/#managers). macOS 및 Linux의 경우 *rbenv*는 Ruby 환경을 관리하는 가장 인기 있는 방법 중 하나입니다.

*fastlane*은 Ruby 버전 2.5 이상을 지원합니다. 사용 중인 Ruby 버전을 확인합니다:

```sh
$ ruby --version
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
```

#### Bundler

*fastlane*에 대한 종속성을 정의하려면 *Bundler* 및 `Gemfile`을 사용하는 것이 좋습니다. 이렇게 하면 사용할 *fastlane* 버전과 해당 종속성이 명확하게 정의되고 *fastlane* 실행 속도도 빨라집니다.

- `gem install bundler`를 실행하여 *Bundler*를 설치합니다.
- 해당 프로젝트의 루트 디렉터리에 `./Gemfile`을 만듭니다.

```ruby
source "https://rubygems.org"

gem "fastlane"
```

- `bundle update`를 실행하고 `./Gemfile`과 `./Gemfile.lock`을 버전 제어에 추가합니다.
- *fastlane*을 실행할 때마다 `bundle exec fastlane [lane]`을 사용하십시오.
- CI에서 `bundle install`를 첫 번째 빌드 단계로 추가합니다.
- *fastlane*을 업데이트하려면 `bundle update fastlane`을 실행하세요.

### Homebrew (macOS)

이렇게 하면 Ruby를 별도로 설치할 필요가 없으며 대신 *homebrew*가 *fastlane*에 가장 적합한 Ruby 버전을 설치합니다. 자세한 내용은 [이 페이지](https://formulae.brew.sh/formula/fastlane)를 참조하세요.

```sh
brew install fastlane
```

### System Ruby + RubyGems (macOS/Linux/Windows)

이것은 로컬 환경에 권장되지 않지만 시스템 Ruby 환경에 *fastlane*을 설치할 수 있습니다. `sudo`를 사용하면 나중에 파일 권한으로 인해 원치 않는 결과가 자주 발생하고 환경 관리가 더 어려워집니다.

```sh
sudo gem install fastlane
```

## fastlane 초기화

```
fastlane init
```

프로젝트 디렉토리에서 이 명령어를 실행해주면 fastlane 스크립트가 생성됩니다.

## fastlane 으로 터미널 명령어 실행하기

```ruby
lane :lint_pod do
    Action.sh("pod spec lint --verbose ../UniversalSDK.podspec")
end
```

## framework 빌드 및 배포하기

### git 명령어 사용하기

[fastlane docs for iOS Setup](https://docs.fastlane.tools/getting-started/ios/setup/)

## Jenkins 사용시 설정

iOS 빌드시 환경변수 설정

Jenkins 관리 - 시스템 설정 - Global properties

```
key : LANG
val : en_US.UTF-8

key : LC_ALL
val : en_US.UTF-8

key : Path
val : /bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```
m1 일경우
```
key : Path
val : /bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin
```