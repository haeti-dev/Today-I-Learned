https://github.com/nektos/act

## Act

- Github Actions를 로컬에서 구동할 수 있게 해주는 CLI 툴
- 워크플로우 파일(.github/workflows/*.yml)에 작성된 Job과 Step들을 Docker 컨테이너 안에서 실행한다.

## 활용법

- `act [이벤트명] [옵션]`  와 같은 명령어로 특정 이벤트를 트리거할 수 있다. (ex. push, pull_request)
- 이벤트 페이로드(PR 번호, 변경된 파일 목록 등)에 따라 달리 동작하기도 한다.
- 위 경우 JSON 파일을 제공하면 그 파일을 이벤트 페이로드로 사용한다.
- 실제 Github Actions의 runs-on: ubuntu-latest 환경처럼 Docker 컨테이너에서 테스트가 가능하다.
- 하지만 완벽하게 Github 환경을 구성하기는 어려우므로 최종 테스트는 필요하다.

<br>

### 가이드 문서
https://nektosact.com/
