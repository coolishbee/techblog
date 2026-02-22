# AGENTS.md

## 1) 목적과 범위
- 이 문서는 `/Users/james/Documents/workspace/techblog`에서 동작하는 에이전트용 작업 규칙이다.
- 저장소의 주 목적은 MkDocs 기반 기술 문서 사이트 운영이다.
- 핵심 수정 대상은 `docs/**/*.md`, 설정은 `mkdocs.yml`, 템플릿 오버라이드는 `overrides/**`이다.
- 생성 결과물 디렉터리 `site/`는 빌드 산출물로 간주한다.

## 2) 저장소 구조 요약
- `mkdocs.yml`: 사이트 네비게이션, 테마, markdown extension 설정의 단일 기준점
- `docs/`: 실제 문서 본문
- `overrides/`: Material 테마 커스터마이징(Jinja/HTML)
- `README.md`: 최소 소개 파일
- `site/`: 정적 빌드 출력

## 3) Cursor/Copilot 규칙 반영 상태
- `.cursor/rules/` 확인 결과: 없음
- `.cursorrules` 확인 결과: 없음
- `.github/copilot-instructions.md` 확인 결과: 없음
- 따라서 본 파일의 규칙을 우선 적용한다.

## 4) 개발 환경 준비
- 이 저장소에는 `pyproject.toml`, `requirements*.txt`, `package.json`이 없다.
- 문서 빌드를 위해 Python 가상환경을 먼저 만든다.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install mkdocs mkdocs-material pymdown-extensions
```

## 5) Build / Lint / Test 명령
- 별도 테스트 프레임워크(pytest/jest 등)는 현재 설정되어 있지 않다.
- 품질 게이트는 `mkdocs build --strict`를 기본으로 사용한다.

### 5.1 전체 빌드
```bash
mkdocs build --clean
```

### 5.2 엄격 검증(권장)
```bash
mkdocs build --strict
```

### 5.3 로컬 개발 서버
```bash
mkdocs serve -a 127.0.0.1:8000
```

### 5.4 빠른 검증 루프
```bash
mkdocs build --strict && mkdocs serve -a 127.0.0.1:8000
```

### 5.5 단일 테스트에 해당하는 실행 방법(중요)
- 단일 테스트 명령은 없다(테스트 러너 부재).
- 대신 아래 절차를 단일 페이지 검증으로 사용한다.
1. 대상 파일 하나만 수정 (`docs/.../target.md`).
2. `mkdocs build --strict` 실행.
3. `mkdocs serve`에서 해당 페이지 URL만 확인.
4. 콘솔 에러/깨진 링크/렌더링 오류를 점검.

## 6) 작업 우선순위
- 문서 변경은 먼저 `docs/`에서 해결한다.
- 구조 변경(nav 추가/이동/이름 변경)은 `mkdocs.yml`를 함께 수정한다.
- UI 동작 이슈가 있을 때만 `overrides/`를 수정한다.
- `site/` 직접 수정은 금지(명시적 요청 제외).

## 7) Markdown 스타일 가이드
- 헤딩은 계층 순서를 지킨다(`#` -> `##` -> `###`).
- 페이지당 하나의 핵심 주제 유지, 불필요한 깊은 헤딩 지양.
- 코드 블록은 반드시 언어 태그 사용(`bash`, `go`, `kotlin`, `swift`, `yaml` 등).
- 문단은 짧게, 절차는 bullet/number list로 작성.
- 내부 문서 링크는 상대 경로를 사용.
- 이미지 경로는 이동 시 즉시 갱신하고 빌드로 검증.
- 기존 Material 문법(`!!!`, tabbed, tasklist) 패턴을 유지.

## 8) 코드 예제 스타일 가이드
- 예제 코드는 실행 가능한 형태로 작성한다.
- 필요한 import/use/include는 생략하지 않는다.
- import는 표준 라이브러리 -> 서드파티 -> 로컬 순으로 정렬한다.
- wildcard import는 언어 관례상 불가피한 경우만 허용한다.
- 들여쓰기는 공백 기준으로 일관되게 유지한다.
- 한 코드 블록에 서로 다른 버전/패러다임을 혼합하지 않는다.

## 9) 포매팅 규칙
- 언어별 기본 포매터 관례를 따른다(`gofmt`, Kotlin/Swift 표준 스타일 등).
- 긴 한 줄은 읽기 가능한 길이로 분리한다.
- trailing whitespace를 남기지 않는다.
- YAML/Markdown 들여쓰기 정합성을 항상 확인한다.

## 10) 타입(Type) 규칙
- 공개 함수/인터페이스 경계에서는 타입을 명시한다.
- nullable/optional 가능성은 코드에서 드러나게 표현한다.
- 의미 없는 `any` 남용을 피한다.
- 반환 타입과 에러 경로가 문서 설명과 일치해야 한다.

## 11) 네이밍 규칙
- 변수/함수명은 의도가 드러나는 이름을 사용한다(`userId`, `accessToken`).
- 클래스/타입은 `PascalCase`, 변수/함수는 언어 관례(`camelCase` 등) 준수.
- 문서 파일명은 기존 저장소 패턴(소문자 + 하이픈) 우선.
- 약어는 팀에서 널리 통용되는 수준만 사용한다.

## 12) 에러 처리 규칙
- 예제 코드에서 예외를 무시하거나 빈 catch를 두지 않는다.
- 실패 시 원인 파악 가능한 메시지/로그를 남긴다.
- fallback을 넣는 경우, 조건과 한계를 함께 설명한다.
- 네트워크/파일/외부 API 예제는 실패 경로를 반드시 포함한다.

## 13) `mkdocs.yml` 수정 규칙
- 새 문서를 추가하면 `nav`에 반드시 등록한다.
- 문서 이동/이름 변경 시 기존 nav 경로를 동시 갱신한다.
- markdown extension 변경은 기존 문서 렌더 영향 범위를 점검한다.
- theme/palette 변경 시 `overrides` 동작과 함께 확인한다.

## 14) `overrides/partials/comments.html` 주의사항
- Jinja 블록 구조(`{% if %}` 등)를 깨지 않도록 수정한다.
- giscus `data-*` 속성의 저장소/카테고리 일관성을 유지한다.
- palette 연동 스크립트 변경 시 라이트/다크 전환을 수동 점검한다.
- 브라우저 콘솔 오류가 발생하면 스크립트 변경을 우선 의심한다.

## 15) 변경 검증 체크리스트
- `mkdocs build --strict` 성공
- 변경 페이지 렌더링 확인(`mkdocs serve`)
- nav 링크/내부 링크/이미지 경로 확인
- 코드 블록 언어 태그/들여쓰기 확인
- `site/` 수동 수정 여부 확인

## 16) 커밋/PR 작성 가이드(에이전트용)
- 한 PR에는 한 가지 목적(문서 내용, nav 구조, 템플릿 수정)을 우선한다.
- 왜 바꿨는지(배경/효과)를 본문에 명확히 적는다.
- 후속 작업이 필요하면 TODO를 명시한다.
- 대규모 이동/리네임은 링크 검증 결과를 함께 남긴다.

## 17) 금지/주의 사항
- 근거 없는 도구 추가를 가정하지 않는다.
- 존재하지 않는 테스트 명령을 문서화하지 않는다.
- 외부 비밀값(API key, token)을 예제에 하드코딩하지 않는다.
- 생성 산출물(`site/`)을 원본 문서처럼 취급하지 않는다.

## 18) 향후 도구 추가 시 업데이트 규칙
- markdownlint/link checker/pytest 등을 도입하면 본 파일을 즉시 갱신한다.
- 특히 "single test" 또는 "single file lint" 명령을 최우선으로 명시한다.
- 설치 명령, 실행 명령, 실패 시 해석 방법까지 함께 기록한다.

## 19) 빠른 명령 모음
```bash
# strict build (기본 검증)
mkdocs build --strict

# clean build
mkdocs build --clean

# local preview
mkdocs serve -a 127.0.0.1:8000
```
