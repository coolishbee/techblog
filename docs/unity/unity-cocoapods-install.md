---
comments: true
---

# Unity iOS용 CocoaPods 설치 가이드

### 개요

Pod 설치를 위해 필요한 **Ruby 설치 및 설정**, 그리고 **CocoaPods 설치 과정**을 설명합니다.

---

### 🍎 macOS에서 rbenv로 Ruby 버전 관리하기

#### 1. macOS 기본 Ruby의 한계

macOS에는 시스템 Ruby가 기본으로 포함되어 있지만 다음과 같은 제약이 있습니다.

- `fastlane`, `bundler` 등 설치 시 **sudo 권한 요구**
- 프로젝트별 Ruby 버전이 다를 경우 충돌 발생
- 삭제·재설치를 반복으로 인한 환경 오염

!!! tip "해결 방법"
    **rbenv**를 사용하면 프로젝트별로 Ruby 버전을 독립적으로 관리할 수 있습니다.

---

#### 2. rbenv 설치

```bash
# rbenv 및 ruby-build 설치
brew install rbenv ruby-build

# 설치 확인
rbenv versions
```

출력 예시:

```
* system (set by /Users/...)
```

!!! note
    Homebrew가 이미 설치되어 있다는 가정하에 진행합니다.

---

#### 3. 설치 가능한 Ruby 버전 확인 및 설치

```bash
# 설치 가능한 버전 목록 확인
rbenv install -l

# 특정 버전 설치
rbenv install {version}
```

---

#### 4. 설치된 Ruby 확인 및 버전 전환

```bash
# 설치된 Ruby 목록 확인
rbenv versions
```

예시:

```
* system (set by /Users/username/.rbenv/version)
  3.2.9
```

Global 버전 변경:

```bash
rbenv global 3.2.9
```

---

#### 5. 환경 변수 설정

!!! info
    vi 사용법이 익숙하지 않다면 다른 에디터(nano, code 등)를 사용해도 무방합니다.

##### zsh 사용 시

```bash
vi ~/.zshrc
```

아래 내용 추가:

```bash
[[ -d ~/.rbenv  ]] && \
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"
```

##### bash 사용 시

```bash
vi ~/.bash_profile
```

동일 내용 추가:

```bash
[[ -d ~/.rbenv  ]] && \
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"
```

!!! warning "bash 설정이 필요한 이유"
    Unity의 **iOS Resolver**가 내부적으로 bash 쉘을 강제로 사용합니다.  
    bash에 rbenv 설정이 없으면 시스템 Ruby를 사용하게 되어  
    Resolver의 pod 설치가 실패할 수 있습니다.

---

#### 6. 설정된 Ruby 버전 확인

!!! important
    터미널을 완전히 종료 후 다시 실행하세요.

```bash
ruby --version
```

시스템 Ruby가 아닌, rbenv로 설정한 버전이 출력되면 정상입니다.

---

### CocoaPods 설치

!!! prerequisite
    반드시 rbenv로 Ruby가 설정된 상태에서 진행해야 합니다.

1. CocoaPods 설치

```bash
gem install cocoapods
```

2. 설치 확인

```bash
pod --version
```

---

#### 문제 해결

##### pod 명령을 찾을 수 없는 경우

```bash
rbenv rehash
```

##### 여전히 system Ruby가 보이는 경우

- 터미널 재시작
- `which ruby` 경로 확인
- `.zshrc / .bash_profile` 설정 재점검

---

🎉 이제 Unity iOS Resolver에서 정상적으로 CocoaPods를 사용할 수 있습니다.