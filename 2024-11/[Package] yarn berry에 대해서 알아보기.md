# [Package] yarn berry에 대해서 알아보기
Yarn Berry는 Yarn 패키지 매니저의 최신 버전(2.x 이상)으로, Yarn Classic(1.x)의 후임이다.  
Yarn Berry는 근본적인 구조 변화를 포함하고 있어 기존 Yarn과 다소 다른 방식으로 동작하며, 다양한 장점과 일부 한계가 존재한다.

## Yarn Berry가 나오게 된 역사적 배경
Yarn은 원래 JavaScript 커뮤니티에서 성능과 보안 문제를 개선하기 위해 등장한 npm의 대체 패키지 매니저다.  
하지만 Yarn 1.x (Yarn Classic)은 다음과 같은 문제를 가지고 있었다:

### 의존성 복잡도 증가에 따른 속도 문제
- `node_modules`내에 결국 파일이 많아지면 탐색하는데 많은 시간이 들게 될 수 밖에 없음.

### 유령 의존성(Panthom Dependency) 문제
유령 의존성 문제는 모노레포(monorepo) 구조에서, 호이스팅으로 인해 한 프로젝트에만 설치된 패키지가 다른 프로젝트에서도 사용 가능하게 되는 현상.
![스크린샷 2024-11-15 오전 1.53.43.png](assets%2F%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-11-15%20%EC%98%A4%EC%A0%84%201.53.43.png)

## Yarn Berry의 장점
### Plug’n’Play (PnP)
**`node_modules` 폴더를 없애고**, 의존성 관계를 정확히 분석해 효율적으로 로드함.  
JS의 Map 객체를 활용해 의존성 관리함.
이로 인해 디스크 공간을 절약하고, 속도가 빨라짐.  
의존성을 하나의 `.pnp.cjs` 파일에 기록하여, 의존성을 메모리에서 직접 로드하게 되어 디스크 I/O를 줄이고 속도를 개선함.

### Zero-install
`.yarn/cache` 폴더에 의존성을 압축 파일(.zip) 형태로 저장하여, 프로젝트와 함께 버전 관리 시스템(Git 등)에 커밋할 수 있음.
![스크린샷 2024-11-15 오전 9.59.52.png](assets%2F%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-11-15%20%EC%98%A4%EC%A0%84%209.59.52.png)

## Yarn berry가 Yarn classic 보다 빠른 이유
![스크린샷 2024-11-15 오전 9.46.21.png](assets%2F%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-11-15%20%EC%98%A4%EC%A0%84%209.46.21.png)
