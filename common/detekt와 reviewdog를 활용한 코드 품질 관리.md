## 1. detekt란?

[**github.com/detekt/detekt**](https://github.com/detekt/detekt)

detekt는 깔끔한 Kotlin 코드 작성을 위한 정적 코드 분석 도구이다. 잘못된 서식, 안티패턴, 매우 다양한 룰셋 설정을 제공하며 레거시 프로젝트의 기존 이슈는 무시하되, 새로운 이슈 발생을 방지하는 baseline 또한 생성이 가능하다.

<br>

## 2. reviewdog란?

https://github.com/reviewdog/reviewdog

reviewdog은 모둔 linter 도구와 쉽게 통합하여 깃허브와 같은 코드 호스팅 서비스에 자동으로 리뷰 코멘트를 게시할 수 있는 방법을 제공한다. reviewdog은 lint 도구의 출력을 사용하여 검토할 패치와 다른 결과가 발견되면 이를 댓글로 게시한다.

<br>

## 3. detekt Rule set

```yaml
complexity:  # 코드 복잡도 경고
formatting:  # 코드 서식 경고
naming:      # 네이밍 규칙
performance: # 성능 경고
style:       # 코딩 스타일 교정
coroutines:  # 코루틴 규칙
comments:    # 주석 규칙
potential-bugs: # 잠재적 버그 위험 검사
```

위와 같이 다양한 룰셋 설정이 가능하며, 대부분의 기초 설정은 아래 공식 코드를 사용하고, 필요한 몇 가지만 변경하면 된다.

https://github.com/detekt/detekt/blob/v1.23.7/detekt-core/src/main/resources/default-detekt-config.yml

<br>

## 4. GitHub Actions

detekt로 검사해서 나온 이슈를 reviewdog를 사용하여 리뷰를 받기 위해 GitHub Actions 워크플로우를 생성해보았다.

<br>

```yaml
name: PR Checker

on:
  pull_request:
    branches: [ "dev" ]
  push:
    branches: [ "dev" ]

permissions:
  contents: read
  pull-requests: write

jobs:
  build:
    name: Check Code Quality and Build
    runs-on: ubuntu-latest

    steps:
      
      # 생략 
      - name: Run detekt
        run: ./gradlew detekt
      
      - name: Setup reviewdog
        uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest

      - name: Run reviewdog for app
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: cat ./app/build/reports/detekt/detekt.xml | reviewdog -f=checkstyle -name="detekt-app" -reporter="github-pr-review" -level="info" -tee
```

<br>

- reviewdog가 리뷰를 달기 위해서는 권한이 필요하니 허용하자.
- reviewdog의 최신 버전을 설치한다.
- 먼저 detekt를 실행하고, detekt가 분석한 결과가 있는 파일의 경로를 reviewdog에게 등록한다.
- reporter를 “github-pr-review”로 설정한다. 이를 통해 GitHub PR에 코드 라인별 리뷰 코멘트를 남길 수 있으며, 이를 허용하지 않고 Action에서만 리뷰 받도록 설정할 수도 있다.
- 보고할 이슈의 level을 설정할 수 있다.

<br>

## 결과
<img src="https://github.com/user-attachments/assets/4df8333b-3da2-43a2-b144-a8b399587d2b" width=1000>
